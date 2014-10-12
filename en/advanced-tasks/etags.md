# ETag Everything, Everything ETagg'ed

The first goal of this guide is to show you how relatively easy adding web caching support to a li3 application is. The second goal is to introduce you to the topic of web caching topic in general, showing what makes it so interesting. 

The beneficial effects of web caching are most often underestimated but adding even just basic support to an existing application a guaranteed win for everybody involved. 

**Embrace the web!**

Web caching is both simple and complex. As we don't want to put the cart before the horse, we start simple first. This article assumes that you have a basic understanding of HTTP and applies several simplifications. A full list is presented is presented at the end to give additional pointers for further reading. 
	
## Entity Tags

_Validation_ is one aspect of web caching. With it comes the aspect of _conditional requests_. A typical request/response flow involving web caching and entity tags might look as simple as in the following chart.

![chart](https://dl.dropbox.com/u/1578287/lithium_advent_caching_flow.jpg)

The _entity tag_ (also _ETag_ as the HTTP header) is a so called cache validator. In contrast to other validators like _Last-Modified_, entity tags prove to be **very flexible**. They are perfect for nearly any aspect of an application, generating both dynamic and static content: _a typical web application_. 

Entity tags are like fingerprints of the underlying resource of an URL and change when its content changes. They must only be unique in the scope of that URL. The tag itself can practically be any kind of string - most often this is a hash over _some_ part of the resource.

Now it is important which parts of the resource we choose to generate the hash over. As a last resort this could be as close to the response as computing the hash over the actual body to be sent. Or - and this is much better as we can move the decision of not returning the full response up - generate the hash over the data that is used in rendering the body. Sometimes this work has already been done for you: reuse existing checksums, unique IDs or other modification signals.

In the following three sections we'll visit three different parts of a li3 application and apply basic web caching for: serving files, serving dynamic content, retrieving arbitrary data from a web service.

## Serving Files

One of the benefits when using MongoDB is that you also get a nice place to store your files. A good example on how and why to do this is [Nate's photobolog tutorial](https://github.com/nateabele/photoblog). This code presented here actually builds off the ideas presented in that tutorial.

While there are many good things that come with storing you files this way, one downside is that in order to respond to file requests now involves PHP and the database in addition to the web server. To compensate this overhead we'll modify the route handler used for serving the files to use web caching.

We setup our `Files` model to use GridFs and also add a handy but stubbed `mimeType()` [instance method](/docs/manual/models/adding-functions-to-models.md). You really want to implement real MIME-type detection here. Either go directly with the [fileinfo extension](http://php.net/manual/en/intro.fileinfo.php) or and alternative solution like [the mm\Mime\Type class](https://github.com/davidpersson/mm/blob/master/src/Mime/Type.php).

```php
// models/Files.php

class Files extends \lithium\data\Model {
	protected $_meta = array('source' => 'fs.files');

	public function mimeType($entity) {
		return 'image/png';
	}
}
```

Now this is where we actually serve the file. We'll be reusing the `md5` field as the entity tag. MongoDB automatically adds that field as the checksum over the contents of the file. The entity tag is **wrapped in quotes** so be sure to strip them off before comparing. Also we must include the entity tag on **both** 200 and 304 responses.

```php
// config/bootstrap/routes.php

use lithium\net\http\Router;
use lithium\action\Response;
use app\models\Files;

Router::connect('/files/{:id:[0-9a-f]{24}}.{:type}', array(), function($request) {
	if (!$file = Files::first($request->id)) {
		return new Response(array('status' => 404));
	}
	$response = new Response();
	
	$hash = $file->md5;
	$condition = trim($request->get('http:if_none_match'), '"');
		
	$response->headers['ETag'] = "\"{$hash}\"";

	if ($condition === $hash) {
		$response->status(304);
	} else {
		$response->headers += array(
			'Content-Length' => $file->file->getSize(),
			'Content-Type' => $file->mimeType(),
		);
		$response->body = $file->file->getBytes();
	}
	return $response;
});
```

As you will see the pattern of generating, adding and comparing the entity tag will not change much throughout the next example below and basically stays the same.

## Serving Dynamic Content

This example will enable caching for any possibly dynamically generated content by adding a filter to `Dispatcher::run()`. While this is a pretty bulletproof it is also not the most performant solution as we are generating the entity tag over the full body which in turn needs to be always rendered. 

A better solution would be to generate the tag over the data used in the body (see above) and intercept the cycle right before the template is fully rendered. Feel free to post your solution in the comments.

```php
// config/bootstrap/action.php

use lithium\action\Dispatcher;

Dispatcher::applyFilter('run', function($self, $params, $chain) {
	$request  = $params['request'];
	$response = $chain->next($self, $params, $chain);

	$hash = md5($response->body());
	$condition = trim($request->get('http:if_none_match'), '"');
	
	$response->headers['ETag'] = "\"{$hash}\"";

	if ($condition === $hash) {
		$response->status(304);
		$response->body = array();
	}
	return $response;
});
```

## Interacting with a Service

In this case we are retrieving the latest commits from the li3 GitHub repository. As GitHub has a pretty tight rate limit in place this seems to be a good idea.  

```php
use lithium\data\Connections;
use lithium\storage\Cache;

Connections::add("github", array(
   'scheme' => 'https',
   'type' => 'lithium\net\http\Service',
   'host' => 'api.github.com'
));
$github = Connections::get("github");
$path = 'repos/unionofrad/lithium/commits';

$cacheKey = 'api_github_com_' . str_replace('/', '_', $path);

$options = array('return' => true);
if ($cached = Cache::read('default', $cacheKey)) {
	$options['headers']['If-None-Match'] = "\"{$cached['condition']}\"";
}
$response = $github->get($path, array('type' => 'json'), $options);

if ($response->status('code') == 304) {
	return $cached['body'];
}
$condition = trim($response->headers['ETag'], '"');
$body = $response->body();

Cache::write('default', $cacheKey, compact('condition', 'body'));
return $body;
```

This time we act as the client to the GitHub service, which now has to handle all the entity tag generation and validation. On the other hand we are burdened with the task of providing persistent caching of the response and the entity tag that comes with. 

## Sources

* [RFC 2616 - Caching in HTTP](http://tools.ietf.org/html/rfc2616#section-13)
* [RFC 2616 - ETag](http://tools.ietf.org/html/rfc2616#section-14.44)
* [RFC 2616 - If None Match](http://tools.ietf.org/html/rfc2616#section-14.26)
* [MongoDB GridFS](http://www.mongodb.org/display/DOCS/GridFS+Specification)
* [Wikipedia on HTTP ETag](http://en.wikipedia.org/wiki/HTTP_ETag)
* [Design for the web](http://www.dehora.net/journal/2007/07/earned_value.html)
* [About: Design for the web](http://www.tbray.org/ongoing/When/200x/2007/07/31/Design-for-the-Web)
* [Pitfalls of ETags](http://www.mnot.net/blog/2007/08/07/etags)
* [Things caches do](http://tomayko.com/writings/things-caches-do)
* [Controlling Caches/Tutorial](http://www.mnot.net/cache_docs/)
* [PHP Fileinfo](http://php.net/manual/en/intro.fileinfo.php)
* [MM - the PHP media library](https://github.com/davidpersson/mm)
* [RFC 2616 - Vary](http://tools.ietf.org/html/rfc2616#section-14.44)
* [RFC 2616 - Cache-Control](http://tools.ietf.org/html/rfc2616#section-14.19)
* [How to compare Weak Entity Tags](http://www.w3.org/Protocols/HTTP/1.1/rfc2616bis/issues/#i71)
* [Problems with Entity Tags On Write](http://greenbytes.de/tech/webdav/draft-reschke-http-etag-on-write-latest.html)

