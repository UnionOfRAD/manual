# Routing HTTP Requests

## Introduction

li3's routing allows developers to completely decouple the application's URLs from it's underlying structure. It works by creating a set of `Route` objects that tell li3 to respond to incoming requests, and which bits of code they relate to in your application. While this makes for great SEO and usability, it also keeps things nimble in respect to change.

As such, the router has two main responsibilities. First, to determine the correct set of dispatch parameters based on an incoming request. Secondly, to generate URLs from a given set of parameters.

Though this section's main focus is to show you how to create routes according your needs, we'll also cover how the router builds URLs based on parameters you supply.

## Defining Routes

Defining routes is done in the application directory at `/config/routes.php`, by using the `Router::connect()` method to create `Route` objects that define URL-to-code mappings.

<div class="note note-info">
	The router will match routes in the order they are defined. In other words, the first route that matches will be returned and used for dispatching.
</div>

### Routing Definition Example

Let's start with a simple example: connecting a URL with a controller method:

```php
// The following lines are equivalent...
Router::connect('/help', array('controller' => 'Users', 'action' => 'support'));
Router::connect('/help', 'Users::support');
```

If your application was hosted at http://www.example.com, requesting http://www.example.com/help would show you the rendered results of the `support()` action of `UsersController` in your application. 

### Params & Regex

While helpful, you'll quickly run into situations where something a bit more complex is needed. Most routes in an application include dynamic parameters that are handed to the controller. These parameters are marked in route definitions using the `{:paramname}` syntax. Consider the following example from an application that shows Basketball game rosters:

```php
Router::connect('/{:controller}/{:action}/{:gameId}/{:playerId}', 'Rosters::view');
```

This action forwards the users on to the `view()` method of `RostersController`, and sets the corresponding params on the request so they're available in the controller (`$this->request->params['gameId']` and `$this->request->params['playerId']`, in this case).

Apart from allowing users to supply those values, you can also supply them statically in a route:

```php
Router::connect('/socks', array('Products::view', 'id' => 72739));
```

In order to avoid overlapping cases and provide routing clarity, you can also specify a route parameter with an accompanying regular expression. Similarly defined routes use the `{:paramname:regex}` syntax. There are a few examples in the default `routes.php` file that ships with li3:

```php
Router::connect('/{:controller}/{:action}/{:id:\d+}');
```

Here, we're routing incoming requests along to their respective controllers and actions, but also tacking on a new parameter "id" if the URL ends with a numerical component. The regex here is important. If not defined, this route would also match `/products/viewCategory/electronics` if defined before another route that matches it better.

### Default Parameters

There are a number of default parameters that li3 is aware of. As you build your routes, keep these routes in mind, as they're reserved for routing/dispatching purposes:

	- `controller` : The name of the controller to dispatch.
	- `action` : The name of the action to call in the dispatched controller.
	- `type` : Used for media type routing (covered in the Controllers guide).
	- `args` : Used for continuation routes.

### Continuation Routes

Continuation routes are a new class of route definitions that wrap other routes. They're especially handy if you're used to using some sort of route prefixes to define a state in your application. Such uses may include:

 - Localization
 - Administrative sections of the application
 - API endpoints

This is done by using the special `{:args}` parameter and setting the `continue` parameter to `true`. Once this is defined, you can allow later routes to match as needed. Here's a simple example to wrap your application's URLs according to locale:

```php
Router::connect('/{:locale:en|de|it|jp}/{:args}', array(), array('continue' => true));
```

As you can see, this route tells li3 that routes that are prefixed with 'en', 'de', 'it', or 'jp' should set an additional `locale` request parameter then be passed back to the router for further matching. A few other examples:

```php
// API endpoint versioning (i.e. /v1/products/list.json)
Router::connect('/{:version:v\d+}/{:args}', array(), array('continue' => true));

// Admin routing...
Router::connect('/admin/{:args}', array(), array('continue' => true));

// For rendering all static pages...
Router::connect('/pages/{:args}', 'Pages::view', array('continue' => true));
```

### Route Matching

li3's router is also used in reverse: instead of turning URLs into parameters (controllers and actions, at least) it can also create application URLs based on supplied parameters based on the defined routes.

Usually you'll be using this functionality without realizing it. For example, it's used by the `Html` helper in views to create links. Normally it's faster and easier to use the supplied helper functions. If however you're doing something in a layer that doesn't have easy access to this functionality, you can use the router directly.

Full details are supplied in the API docs, but the basic idea is that you can use `Router::match()` to do this. Just supply a set of parameters, and the router will return a URL (if any) that matches that set of parameters:

```php
// Imagine this route has already been defined:
Router::connect('/unicorns', 'Ponies::magic');

Router::match(array('controller' => 'Ponies', 'action' => 'magic'));
// Returns '/unicorns'
Router::match('Ponies::magic');
// Also returns '/unicorns'
```
