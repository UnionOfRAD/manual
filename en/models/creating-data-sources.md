# Creating Data Sources

## Introduction

If you're reading this guide, chances are you've looked around for a data source in the li3 community but haven't yet found it. Whether it's LDAP or the Flickr API you want to connect to, you're in luck. This guide will walk you through the thinking and the process necessary to create your own data source. First, we'll discuss some of the overall architectural knowledge needed, then we'll dive into the details with a real example.

## Metadata Methods

When creating a data source, it's important to realize the role of a data source. Data sources provide a layer that assists models, but focuses on the details surrounding connecting to, authenticating, and facilitating generic reads and writes to a certain data store. When creating a data source, work on the generic tasks needed to get the work done rather than the domain-specific work models will perform later on.

As such, it's important for li3's models to understand the structure of the underlying data. A well-written data source should provide that information in a format models understand.

There are two main metadata methods your data source will want to implement &mdash; `sources()` and `describe()`:

<table>
	<thead>
	<tr>
		<th>Method Name</th>
		<th>Question it Answers</th>
		<th>RDMBS Example</th>
		<th>NoSQL Example</th>
		<th>API / Service Example</th>
	</tr>
	</thead>
	<tbody>
	<tr>
		<td><pre>sources()</pre></td>
		<td>What objects can models bind to?</td>
		<td>A list of tables in the database.</td>
		<td>A list of collections in the database.</td>
		<td>A list of resource collections available in a REST API.</td>
	</tr>
	<tr>
		<td><pre>describe()</pre></td>
		<td>What properties does each entity have?</td>
		<td>What columns are in a given table, and what are their types?</td>
		<td>May only return an empty array, as the schema could be different between documents.</td>
		<td>Which properties are mutable/searchable on the objects in the API, and what type are they?</td>
	</tr>
	</tbody>
</table>

Focus on understanding the meaning behind these methods for now—we'll be implementing them and discussing the details as we create a real data source shortly.

## Queries and Entities

Once models understand the basic shape for your data, the next step is facilitating communication between the data source and its associated models.

In li3, models ask the data source questions through [`Query` objects](http://li3.me/docs/lithium/data/model/Query), and the data source (typically) answers with one or more `Entity` objects, which are either a `Record` or a `Document`. For example, when a model is to find data, it packages up a `Query` object that contains structured data about what sort of information the model is asking for, in what order it expects it returned in, paging information, etc.

It passes this `Query` to the data source, which connects and authenticates to the data store. It inspects the query to see what the model needs, interacts with the underlying data store, and wraps the response data in some sort of `Entity`. If the response requires more than one `Entity`, it's usually wrapped up in a `Collection` like a `RecordSet` or `DocumentSet`.

Once the model has the `Entity` (or collection of entities), it can provide that information back to the caller (i.e. a controller).

## Example Data Source: GitHub Issues API

Let's dive into a real-life example of creating a data source. For the remainder of this guide, we'll walk through the process of creating a new data source that connects to, authenticates with, and moves data to and from the GitHub API. Version 2 of the GitHub API is available via a simple RESTful interface that uses HTTP authentication. We'll be setting up a new data source, using the existing [`data\source\Http` base class](http://li3.me/docs/lithium/data/source/Http) as a starting point.

### Setup

The end result we're looking for is being able to create models that read from and write to GitHub using our data source. Let's get started by creating the data source file and creating a connection to it. First, create a new file in `extensions/adapter/data/source/http/GitHub.php`, inside your app or plugin:

```php
namespace app\extensions\adapter\data\source\http;

use lithium\util\String;
use lithium\util\Inflector;

class GitHub extends \lithium\data\source\Http {}
```

This puts the class in the proper namespace, and imports some utility classes we'll be using later on.

 _A note on organization:_ Most data source adapters break down into a few basic _types_. For example, all relational databases are of the `database` type. They all work similarly, all share a common base class, and all live in `data\source\database` (in core), or `extensions\adapter\data\source\database` (in an application or plugin). Likewise with data sources that connect to databases or web services over HTTP. However, other data sources like [ `MongoDb`](http://li3.me/docs/lithium/data/source/MongoDb) don't have a type. This distinction becomes important when configuring them, as seen below.

Now, let's create a connection that uses this new data source. Add a few lines to `config/bootstrap/connections.php`:

```php
Connections::add('github', array(
	'type'     => 'http',
	'adapter'  => 'GitHub',
	'login'    => 'myusername',
	'password' => '5up3Rs3CrE7',
	'token'    => 'e83fd93a7e099c8523ab99d003ce939a',
));
```

Since the GitHub API allows for an API token to be passed instead of a password, we've added that to the connection information.

Finally, we'll need to create a new model and controller to test the functionality as we build it:

```php
namespace app\models;

class Issues extends \lithium\data\Model {

	public $_meta = array('connection' => 'github');
}
```

These can be saved in `models/Issues.php`, and `controllers/IssuesController.php`, respectively.

```php
namespace app\controllers;

use app\models\Issues;

class IssuesController extends \lithium\action\Controller {

	public function index() {}
}
```

### Connecting & Authenticating

Now that we've got the basic classes in place, let's make sure that the model can connect to GitHub API correctly. Looking at the HTTP data source we're extending, we can see that the configuration details for the `lithium\net\http\Service` object are passed to the data source's constructor, along with the connection details. By overriding the constructor, we can supply our own GitHub-specific connection details:

```php
public function __construct(array $config = array()) {
	$defaults = array(
		'scheme'   => 'https',
		'host'     => 'github.com',
		'port'     => 443,
		'login'    => null,
		'password' => null,
		'token'    => null,
		'auth'     => 'Basic',
		'version'  => '1.1',
		'basePath' => '/api/v2/json',
	);
	$config += $defaults;

	if ($config['token']) {
		$config['login'] = $config['login'] . '/token';
		$config['password'] = $config['token'];
	}
	parent::__construct($config + $defaults);
}
```

What we're doing here is checking to see if there's a token related to the connection. If so, we use it and adjust the username and password field as noted on the GitHub API website. Otherwise, we'll just pass up the configuration to the parent constructor, which will set the configuration options for the new `Service` object with the details we need. As we make changes, we're using the `$config` parameter of the data source. This constructor parameter is populated with the same data we passed into `Connections::add()`.

### Reads

Next, let's use our model to read from the data source. This is done by the model's `find()` method. Let's place a call to `find()` inside our controller, and supply a few parameters the GitHub API is going to need to fetch some issues from a project:

```php
public function index() {
	$results = Issues::find('all', array(
		'conditions' => array(
			'user' => 'myuser', 'repo' => 'myrepo', 'state' => 'open'
		)
	));
}
```

If you try this, `$results` will get filled with the 404 response from GitHub's servers. This is because the default behavior of `read()` doesn't fit our use-case. Let's override it in our new data source in order fix that. Remember that the model layer communicates to the data source via `Query` objects. The first argument for `read()` is the `Query` from our `Issues` model. We can inspect that to find out what the model wants, and translate that into HTTP requests to the GitHub API.

Let's override `read()`, and inspect the `Query` object so we can direct the query to the GitHub API properly.

```php
public function read($query, array $options = array()) {
	$paths = array(
		'issues' => '/issues/list/{:user}/{:repo}/{:state}'
	);

	$params = $query->export($this, array('keys' => array('source', 'conditions')));
	$source = $params['source'];
	$conditions = (array) $params['conditions'] + array(
		'user' => '', 'repo' => '', 'state' => ''
	);

	if (!isset($paths[$source])) {
		return null;
	}
	$path = String::insert($paths[$source], array_map('urlencode', (array) $conditions));
	return json_decode($this->connection->get($this->_config['basePath'] . $path), true);
}
```

First, what we're doing is defining an array that maps resource names to API URLs. This makes our design more extensible, as we can come back later and map other API resources.

Next, we're asking the `Query` object to export the parameters that we care about for this API call, by passing it the adapter instance (`$this`) along with the list of values we care about: `'source'` being the API resource accessed (`'issues'`), and `'conditions'`, the array of conditions values we passed to `find()`. We then assign these to local variables, making sure that the default keys we care about for `$conditions` are initialized.

After that, we'll check to make sure the resource we're attempting to read from is one we know about in our map, and bail out if not. In this case, our `Issues` model is bound to the `'issues'` resource, which happens automatically by naming convention. We then craft the URL to the GitHub API using the conditions from the `Query`.

At this point, you've got a model that now returns array data that matches the JSON data returned from the GitHub API, but let's take this a bit further. We'll want to take advantage of li3's data objects: in this case, the `DocumentSet` and `Document` objects. Doing so allows your application or plugin to present an API that's consistent with li3's data APIs, and can be handled uniformly in controllers and templates.

We'll format the data for use in the model using two important data source methods: `item()` and `cast()`. The `item()` method is used to create data objects, and `cast()` is used by the data source to walk recursively through data structures and format them as you specify.

Both methods use li3's dependency injection mechanism to know what classes to use. As such, it's important to understand it at a basic level. Rather than a complex class structure of managers or containers, li3 uses class properties to manage class dependencies. If you look at the `$_classes` property of a li3 object, you'll see the types and fully namespaced class paths of each dependency. In our case, we want to define what `item()` uses to create data objects. Let's add a `$_classes` definition to our data source:

```php
protected $_classes = array(
	'service' => 'lithium\net\http\Service',
	'entity'  => 'lithium\data\entity\Document',
	'set'     => 'lithium\data\collection\DocumentSet',
);
```

The `'service'` dependency is included in the array because it's needed for the parent `Http` data source class, but it's something we could redefine if needed. The most important points to note are the last two entries: `'entity'` and `'set'`. By defining these dependencies, we're telling the data source what classes to use to package our data, either in singular or plural form. Because GitHub API data responses are JSON objects (that sometimes include embedded objects), we'll use `Document` rather than `Record`.

Once that's in place, we can adjust our `read()` function to return the results of an `item()` call:

```php
public function read($query, array $options = array()) {
	$paths = array(
		'issues' => '/issues/list/{:user}/{:repo}/{:state}'
	);

	$params = $query->export($this, array('source', 'conditions'));
	$source = $params['source'];
	$conditions = (array) $params['conditions'] + array(
		'user' => '', 'repo' => '', 'state' => ''
	);

	if (!isset($paths[$source])) {
		return null;
	}
	$path = String::insert($paths[$source], array_map('urlencode', (array) $conditions));
	$result = json_decode($this->connection->get($this->_config['basePath'] . $path), true);

	return $this->item($query->model(), $result[$source], array('class' => 'set'));
}
```

The `item()` method needs to know what model the data is coming from, the data itself, along with some options. Here, we're passing along an option that specifies that we want the data we're supplying returned as our defined `'set'` dependency. In this case, we've configured that as `lithium\data\collection\DocumentSet`.

Running `find()` on our `Issues` model now gives us a `DocumentSet`, but if we inspect a specific entry in the set (by iterating, for example) we see that the entities in the set are just plain PHP arrays. Let's add the final piece to our data source that will create `Document` objects out of our responses.

The `cast()` method is used by the data source to recursively inspect and transform data as it's placed into a collection. In this case, we'll use `cast()` to transform arrays into `Document` objects:

```php
public function cast($entity, array $data, array $options = array()) {
	$model = $entity->model();

	foreach ($data as $key => $val) {
		if (!is_array($val)) {
			continue;
		}
		$data[$key] = $this->item($model, $val, array('class' => 'entity'));
	}
	return parent::cast($entity, $data, $options);
}
```

Notice how we're using `item()`, but to create entities rather than sets now? Here's where our `'entity'` dependency comes into play. Also note that because `cast()` recursively inspects data, arrays embedded into our GitHub API responses will also be cast as `Document` objects.

At this point, your `Issues` model should be successfully connecting to the API and fetching lists of issues and returning that data in `Documents` inside of `DocumentSet` objects.

### Creating and Updating

Now that we're rocking with some reads, let's hook up the writes. This is done by POSTing some data to the GitHub API. First, let's change our controller so that the model is saving a new issue:

```php
public function index() {
	$issue = Issues::create();
	$issue->user = 'myuser';
	$issue->repo = 'myrepo';
	$issue->title = 'Title for my new issue!';
	$issue->body = 'Here\'s a description of the issue.';

	$issue->save();
}
```

The API specifies that we POST to a URL composed of the user and repository names, and includes data about title and body. We'll need to make a new `create()` method in our data source that assembles the right URL and POST data based off what it receives from the model. In this case, `create()` is handed a `Query` that has an `Entity` we can inspect and use.

```php
public function create($query, array $options = array()) {
	$paths = array(
		'issues' => '/issues/open/{:user}/{:repo}'
	);

	$params = $query->export($this, array('source', 'data'));
	$source = $params['source'];
	$data = array_map('urlencode', (array) $params['data']['data']);

	if (!isset($paths[$source])) {
		return false;
	}
	$path = $this->_config['basePath'] . String::insert($paths[$source], $data);

	switch ($source) {
		case 'issues':
			$data = $query->entity()->data();
			$data = array_intersect_key($data, array('title' => '', 'body' => ''));
		break;
	}
	$result = json_decode($this->connection->post($path, $data), true);
	$key = Inflector::singularize($source);
	return isset($result[$key]);
}
```

As before, we'll create a map of available resources to API endpoints, and then extract out the parameters that we need from the query. _Note_: Sometimes you need lots of parameters, or you might not know all the parameters you need right at the beginning. You can simply omit the second parameter to `export()` to retrieve all of them.

After we've verified the resource we're trying to access is valid, we'll build our API URL just like in `read()`. Next, we'll pull out the data which is to be submitted via the POST request, and whitelist it so we're only pulling the `'title'` and `'body'` fields.

Since the GitHub API returns a JSON object on success, we inspect the response to make sure it's there, and return a boolean based on the success of the operation. This works fine, and our controller can check the value of `save()` to ensure that the issue was created successfully.

However, subsequent code won't be able to verify that `$issue` actually exists in the database, and none of our code will be able to reference it within the GitHub API, because it hasn't received a key (or any other information) that the API has given us. To fix this, we can simply change the last few lines of the `create()` method, as follows:

```php
$result = json_decode($this->connection->post($path, $data), true);
$key = Inflector::singularize($source);

if (!isset($result[$key])) {
	return false;
}
$query->entity()->sync(null, $result[$key]);
return true;
```

The first thing to do is, instead of simply returning the result of the key check, just bail out if the operation failed. Next, we call the `sync()` method of the `Entity` class. This method lets our `Document` instance know that it is now current with the data stored in the service backend, and also lets us update the `Document`'s internal state with any new information received.

Because the API does not define an explicit primary key for issues, we simply pass `null` as the first parameter (we could create a primary key by convention within our adapter, but that's another matter). To the second parameter, we pass the entire array of new issue data received from the API, including identifier values like `position` and `number`, the `user` name and `gravatar_id` of the GitHub user that posted it, and meta information like the `created_at` timestamp, the current `state` (`'open'`), and the `html_url` of the issue's page within GitHub.

The other important piece of internal state modified by `sync()` indicates to `$issue` whether or not it _exists_ within the service. Without this, calling `$issue->exists()` would continue to return `false`, even if the call to `save()` were successful. It's up to your data source to manage the state of the entity objects it handles.

Finally, for implementing updates, we can follow essentially the same template we did for the `create()` method, with a couple of notable exceptions:

 - Instead of calling `post()` on our `$connection` property, we call `put()`.
 - When exporting parameters from the `Query` object, we use `$params['data']['update']`, instead of `$params['data']['data']`. This provides us with just the fields that have been updated on the `Document` object.

## Handling Errors

So far, there are two exceptional conditions which we've encountered in our data source. The first is that this data source does not support delete operations. Accordingly, we should override the `delete()` method, so that it raises an exception if called. li3's `data` package provides a generic `QueryException` for failed query operations. We can use it to prevent delete operations from proceeding, and notify other developers that these API calls aren't permitted.

First, import `QueryException` at the top of the data source class:

```php
use lithium\data\model\QueryException; 
```

Then, override the `delete()` method of the parent class, and throw the exception:

```php
public function delete($query, array $options = array()) {
	throw new QueryException("Delete operations are not supported by this API.");
}
```

The other condition which we have so far not dealt with is that of errors generated by the API. If you've been inspecting the JSON data that the GitHub API returns, you'll have noticed that certain API calls return a JSON structure with an `error` key.

To implement error handling for this case, simply add the following check right below where the return value of `json_decode()` is assigned to `$result` in any of our implemented CRUD methods:

```php
if (isset($result['error'])) {
	throw new QueryException($result['error']);
}
```

This will cause any errors generated by the API to bubble up to our application's error-handling.

## Refactoring and Adding Metadata

So far in our implementation, we've just been doing the simplest thing possible to get working functionality. However, that's left us with some repetitive operations and patterns across our methods. A lot of the work has to do with building queries and URLs, and defining things like `$paths` arrays.

We can clean some of this up by extracting code out into properties and helper methods. This will not only make it easier for our adapter to support new features down the road, but it also helps us extract out some metadata that the model layer can use for introspection.

The first step is to extract out the paths for the available operations on each resource:

```php
protected $_sources = array(
	'issues' => array(
		'paths' => array(
			'create' => array(
				'/issues/open/{:user}/{:repo}' => array('user', 'repo')
			),
			'read' => array(
				'issues/show/{:user}/{:repo}/{:number}' => array('user', 'repo', 'number'),
				'issues/list/{:user}/{:repo}/{:label}' => array('user', 'repo', 'label'),
				'issues/list/{:user}/{:repo}/{:state}' => array('user', 'repo', 'state'),
				'issues/list/{:user}/{:repo}/{:state}/{:search}' => array(
					'user', 'repo', 'state', 'search'
				)
			)
		)
	)
);
```

Here, we have one key for each resource our data source will expose (we can add more later), and within that, we have a listing of paths, one for each operation, with each URL matching up to an array of keys required to use each one. Also note that the URLs are nested within a `'paths'` key. This allows us to associate other data with each resource down the road, i.e. a schema indicating the various fields exposed, and their data types.

Next, we'll implement a method that will generate our new service URLs, which will be used by the various CRUD methods:

```php
protected function _path($source, $type, array $params) {
	if (!isset($this->_sources[$source]['paths'][$type])) {
		return null;
	}
	$keys = array_keys($params);
	sort($keys);

	foreach ($this->_paths[$source][$type] as $path => $required) {
		sort($required);

		if ($keys !== $required) {
			continue;
		}
		$params = array_map('urlencode', (array) $params);
		$result = String::insert($path, $params);
		return "{$this->_config['basePath']}/{$result}";
	}
}
```

This method will first check to see if a path is defined for the resource/operation combination requested. It will then iterate over the available paths, comparing the keys required for each path to the keys provided to the method, in an attempt to find a match.

When a match is found, it uses `$params` to generate an API URL, which is returned to the calling method. Now, let's refactor the `read()` method, by removing the string-handling code that was previously used to generate URLs, and replace it with a simple set-and-check block which calls our new `_path()` method:

```php
public function read($query, array $options = array()) {
	$params = $query->export($this, array('source', 'conditions'));
	$source = $params['source'];
	$conditions = (array) $params['conditions'];

	if (!$path = $this->_path($source, __FUNCTION__, $conditions)) {
		return null;
	}
	$result = json_decode($this->connection->get($path), true);

	if (isset($result['error'])) {
		throw new QueryException($result['error']);
	}
	return $this->item($query->model(), $result[$source], array('class' => 'set'));
}
```

The new implementation uses just a few lines to extract the necessary parameters from the `Query` object, then uses the new helper method to generate the correct URL, calls it and decodes the result, then implements the error-handling we saw above. Finally, if all goes well, it returns the results of the method call, wrapped in a set of `Document` objects that can be iterated over.

Now that we've extracted the above information into a class property, we can do other useful things with it such as implement the `sources()` method described at the beginning of this section. This method is intended to return a simple array indicating the list of resources available to bind to:

```php
public function sources($class = null) {
	return array_keys($this->_sources);
}
```

## Conclusions

Hopefully this section has armed you with all the knowledge required to create your own data sources. They're a very powerful tool in the framework, that allow you to model many different types of data from disparate sources in a single, unified way.

Just remember, while you may choose to implement your own conventions on data access for the sake of convenience, data sources should only be concerned with interacting with backend data in abstract terms; high-level domain logic should always be implemented in models, or other classes designed to augment models.
