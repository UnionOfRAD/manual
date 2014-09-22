# A Typical li3 Request

When building an application with li3, it's important to understand how a typical request is handled. This gives you a better idea of the possibilities inherent in the system, and allows you to better troubleshoot any problems that might arise during development. This also gives you a good macro view of what's going on before we dive into how controllers, views, and context works inside li3.

## Initial Request and Bootstrapping

While a li3 application can be configured a number of different ways, the safest is by pointing your web server to the `/app/webroot` folder inside the application. It can be pointed at the root directory of the application, but either way, the request is forwarded on until `/app/webroot/index.php` handles it.

This directory index file does two things: first, it loads up li3's main bootstrap file (and any related bootstrap files therein). Second, it instantiates the dispatcher, and hands its `run()` method a new `Request` object. The `Request` object aggregates all the GET / POST / environment data, and is the canonical source of any of that information throughout the life of the request.

## Request Dispatching & Routing

Once `Dispatcher` has the request, it asks the router to process it. This processing basically matches the request against each configured route, in the order it was defined. Once a match is found, the Router returns parameter information necessary to find a controller class, and dispatch a method call against it.

Once those parameters have been returned, the Dispatcher uses them to instantiate the correct controller object, and invokes it. The invokation logic in each controller informs it on which action to call to handle the response.

## Action Rendering

Each controller action contains a set of business logic to build a facet or interface in your application. It may interact with models or other classes to fetch, validate, sanitize, and process the data.

Once the data is ready for the view layer, the action hands it off either through `$this->set()`, or by returning an associative array. That data is passed, along with information about the response type, to the Media class.

## The Media Class

The Media class facilitates content-type mapping (mapping between content-types and file extensions), handling static assets, and globally configuring how the framework handles output in different formats.

Once the controller action data and request has been passed to Media, it matches the request with the correct response type, and triggers the view layer accordingly. Most often this results in rendering an HTML view (and its elements) inside of an HTML layout.

Media is extremely flexible, however, and the response could result in JSON serialization, XML formatting, or writing out the contents of an audio file to the output buffer. Content type mapping is done by calling `Media::type()`, usually in a bootstrap file.

## Content Output

Finally, the response content, along with any headers, are packed into a response class, which is passed by the controller back up to `Dispatcher`. The dispatcher returns the Response object to index.php, where it is echoed to the output buffer.

This echo writes the headers, and performs a buffered output of whatever content was returned from the Media class.