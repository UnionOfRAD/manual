# The Filter System

This guide is meant as an introductory course in creating filters in applications. Filters are essentially an efficient way of introducing event-driven communication between classes in your application. They allow you to inject bits of logic in the middle of main-line program flow while at the same time keeping the API clean, avoiding tight class coupling and some sort of centralized publish/subscribe system.
What would you need a filter for? Here are some quick examples:

* A _before filter_ that marks the `'created'` column in your database with the current date
* Intercepting certain requests to deliver raw file contents rather than an HTML response
* Wrapping profiling measurements around each dispatched request
* Checking each request to make sure the user has been authenticated
* Logging various events in your application
* Creating a read-through cache filter that encapsulates the process of checking whether a result has been cached, serving a cached result, and writing results to a cache backend

By the end of this guide, you should feel comfortable in your understanding of the theory and best practices behind filters, as well as identifying filterable logic in the framework and creating your own filterable methods.

<div class="note note-version">
	The filtering system was overhauled in framework version
	1.1. This guide uses the new API in its examples and
	descriptions. The pre-1.1 syntax is documented <a href="/docs/api/lithium/1.0.x/lithium/util/collection/Filters">here</a>.
</div>

## Aspect-Oriented Programming

The framework's filter system is inspired by similar concepts in Aspect-Oriented Programming, or AOP. In AOP, an aspect refers to a bit of logic that is scattered throughout an application. Usually, it's a piece of logic that is used in a number of different points in the application, often in completely unrelated systems.

There are a few examples of aspects that you're probably already familiar with. Logging and authentication are two great ones. Both are used in many different parts of an application, and maintaining something so widespread can get tricky unless you're organized. Not only that, those bits of logic aren't really part of the code that makes up your business logic. In other words, they're separate _concerns_.

## Filter Structure

Long story short, a filterable method is created by encapsulating the main logic for a method inside a closure. That closure is then passed to `Filters::run()` to allow for chain management.

Applying a custom filter to a filterable method is done by calling `Filters::apply()` with the appropriate information about the method. The first argument specifies the fully quallified class name, the second the name of the method to filter, and the third argument is the closure that defines what you want the filter to do.

The following sections will illustrate a few examples that should help you understand how both to apply and create filters.

## Applying Filters: Authentication

Since it's likely to be more common, let's tackle applying filters first. The location of the filter application might depend some on what it is you're filtering, but bootstrap files are often a good choice.

Let's imagine for a moment that we're aiming to add a filter to a li3 application that checks for authenticated users. Let's start with a basic filter setup, and place it in `bootstrap/session.php` (if you look at `session.php`, you'll notice there's already some configuration for defining session storage and authentication). The thinking here is that we want to inject a bit of authentication verification logic as the dispatcher receives each request. Since `Dispatcher::run()` is filterable, we'll apply the filter to it:

```php
use lithium\aop\Filters;
use lithium\action\Dispatcher;

Filters::apply(Dispatcher::class, 'run', function($params, $next) {
	return $next($params);	
});
```

This is the basic structure for applying a filter: call `Filters::apply()` with information on the class or object in question, hand it the name of the method you'd like to filter, then a closure that implements your filter logic. 

But let's talk about the parameters involved in the closure:

_The first parameter_, `$params` gives you access to any parameters that original method implementation had access to. The bulk of your logic will often be in accessing or modifying what's in those parameters before allowing the filters chain advance and continue.

_The second and last parameter_, `$next`, is important. It is a function supposed to be called with just `$params` as its only parameter. Once called, the next filter in line will be executed until it reaches the last item in the filter chain: the original method implementation itself. 

Unless you want to short-circuit filter execution (which includes the base logic of the method you're filtering), you'll want to include this somewhere in your filter logic. The position this call has in your filter also might be important. For example, if you're creating a filter you want to have happen __after__ a certain method, your custom filter logic should follow the `next()` call rather than proceed it:

```php
use lithium\aop\Filters;

Filters::apply(SomeClass::class, 'methodName', function($params, $next) {
	// Custom "before" logic goes here.
	$result = $next($params); // if you don't call $next() the chain is short-circuited
	// Custom "after" logic goes here.
	return $result;
});
```

Alright: let's use these parameters to our advantage in creating an authentication setup. If we look at the parameters for `Dispatcher::_callable()` we can see that accepts three: the current `Request` object (an instance of `lithium\action\Request`), the parameters returned from routing the request, and an options array.

When designing filters, it's important to have a clear understanding of the method you're hooking into, and where it fits in the framework's lifecycle. The `_callable()` method is invoked immediately after the application's routes have been processed, and routing parameters have been returned. This method's responsibility is to take those routing parameters and convert them to a callable object (i.e. a controller) that can process the request and turn it into a response.

For this filter, we'll intercept the method after it finds a controller object (assuming it finds one), but before it returns it to `Dispatcher::run()`:

```php
use lithium\aop\Filters;
use lithium\action\Dispatcher;
use lithium\action\Response;
use lithium\security\Auth;

Filters::apply(Dispatcher::class, '_callable', function($params, $next) {
	$ctrl    = $next($params);
	$request = isset($params['request']) ? $params['request'] : null;
	$action  = $params['params']['action'];

	if (Auth::check('default')) {
		return $ctrl;
	}
	if (isset($ctrl->publicActions) && in_array($action, $ctrl->publicActions)) {
		return $ctrl;
	}
	return function() use ($request) {
		return new Response(compact('request') + ['location' => 'Sessions::add']);
	};
});
```

This particular listing assumes you've set up a configuration with the `Auth` class called `'default'`. Check out the authentication chapter for more on this. Taking it from the top, we can see that the filter first continues executing the chain in order to get the `Controller` object (`$ctrl`) to work with, and does all of its processing _after_ the actual method executes.

Next, it uses the `Auth` class to see if a user is logged in, and if so, allows the application to continue normally. Then, it checks to see if the controller has a `$publicActions` property defined, and if so, checks the array of whitelisted actions to see if the requested one matches, and again, allows processing to continue if so.

Finally, if the user is not logged in, and the requested action is not public, the user is redirected to the login page. Note that, as opposed to returning a `Response` object directly, as would be appropriate when filtering other `Dispatcher` methods like `run()` or `_call()`, here we are returning a closure.

It is important when filtering a method that we honor the method's _contract_. That is, when filtering a method, if we intercept and modify its parameters, or if we bypass it and substitute the return value, we must still keep the values within the acceptable range of what the method accepts or produces. In the case of the `_callable()` method, it is expected to produce a callable object. The `Controller` uses PHP's magic `__invoke()` method to achieve this. Therefore, we can interchange controller objects with closures, and vice versa.

Implementing the controller for this example might look something like this:

```php
namespace app\controllers;

use lithium\security\Auth;

class SessionsController extends \lithium\action\Controller {

	public $publicActions = ['add'];

	public function add() {
		if ($this->request->data) {
			$success = Auth::check('default', $this->request);
			// ...
		}
		// ...
	}
}
```

The rest of the code necessary to implement user authentication is covered in another chapter, but take special note of the `$publicActions` property in the controller. Without this, the `add()` action would be treated as private, resulting in an infinite redirect loop.

Although authentication is a very deep subject, through this example you should be able to see how, whatever your app's rules and constraints are, integrating into your application should be straightforward, and most importantly, unobtrusive.

## Applying Filters: Logging

One more example is common enough to cover: logging. When building a large application, it's handy to log SQL queries made against your data store. It helps in debugging and performance optimization. Let's cover building a quick filter around your data connection logic in order to log SQL statements to a file.

The approach here requires a bit of understanding how the data layer works. In the setup of your application, you've probably created new database (or other datasource) connections in `config/bootstrap/connections.php`. Here's what a simple connection to a MySQL database might look like:

```php
Connections::add('default', [
	'type' => 'database',
	'adapter' => 'MySql',
	'host' => 'localhost',
	'login' => 'myusername',
	'password' => 'mypassword',
	'database' => 'appname'
]);
```

Looking at the API of the data source adapters, we want to add logging to the `_execute()` method. Its only parameter is the SQL being executed, and that's exactly what we'll need for our filter logic.

The adapter classes are implemented as concrete classes. That's also why `Connections::get('default')` in the example below will give us an instance of the `MySql` class. Using that we can continue and filter just calls to that particular instance or using a different syntax in `Filters::apply()`, to filter *any* instance of that class.

```php
use lithium\aop\Filters;
use lithium\analysis\Logger;
use lithium\data\Connections;
use lithium\data\source\database\adapter\MySql;

// Set up the logger configuration to use the file adapter.
Logger::config([
	'default' => ['adapter' => 'File']
]);

// Filter any instance of the MySql adapter (preferred):
Filters::apply(MySql::class, '_execute', function($params, $next) {
	// Hand the SQL in the params headed to _execute() to the logger:
	Logger::debug(date('D M j G:i:s') . ' ' . $params['sql']);

	// Always make sure to keep the filter chain going.
	return $next($params);
});

// Filter just that single instance:
Filters::apply(Connections::get('default'), '_execute', function($params, $next) {
	// ...
});
```

## Creating Filter-able Logic

If you're planning on creating and distributing your own code you might consider writing parts of your API to be filter-able. This allows other developers to take advantage of our powerful filter system, while at the same time helping you avoid writing extra callback methods or configuration options in your code.

Let's imagine for a bit that you're creating a social media integration extension for li3. The library will connect to popular social networking sites to post or gather information. Since you're offering this to other developers to use in their applications, it'd be nice to enable filtering on some of the logic.

For example, let's add a filter to the method that posts content to Twitter. It's conceivable that developers might want to filter that logic to shorten URLs using different services, or automatically include hashtags at the end of the tweet.

First, start with the basics:

```php
class Twitter {

	public function tweet($status, $options) {

		$defaults = ['responseFormat' => 'json'];
		$options += $defaults;

		// Twitter connection and data transfer logic goes here...
		return $result;
	}
}
```

Here we've got a basic method implementation. Of note is the `$options` parameter: the framework API is a big fan of keeping parameter counts small using options arrays. Coding to that same standard helps developers more quickly understand how to put things together. Here you can see a bit of prep work happen before the main logic of the method, mostly just in merging some default values into the supplied options.

Once we've got the core logic in, adding the ability to filter it is relatively painless. What we end up doing is wrapping up the core implementation in a closure that we hand to `Filters::run()`, and returning the result of that call. Here's what it looks like:

```php
use lithium\aop\Filters;

class Twitter extends \lithium\core\Object {

	public function tweet($status, $options) {

		$defaults = ['responseFormat' => 'json'];
		$options += $defaults;

		$params = compact('status', 'options');

		return Filters::run($this, __FUNCTION__, $params, function($params) {
			// Twitter connection and data transfer logic goes here...
			return $result;
		});
	}
}
```

The `filter()` method takes four arguments. The first is the object instance (`$this`) or the class' name (`get_called_class()`), the second is the name of the method, the third is an array of parameters the original method implementation takes. The last parameter is a closure containing the original method implementation. That's it! If you're doing something pretty simple - that's all you need to do. 

There are a few situations that make things a bit different. One example is if your method is accessed statically. Let's adjust our `tweet()` example to illustrate (note that the parent class also changes):

```php
use lithium\aop\Filters;

class Twitter extends \lithium\core\StaticObject {

	public static function tweet($status, $options) {

		$defaults = ['responseFormat' => 'json'];
		$options += $defaults;

		$params = compact('status', 'options');

		return Filters::run(get_called_class(), __FUNCTION__, $params, function($params) {
			// Twitter connection and data transfer logic goes here...
			return $result;
		});
	}
}
```

Mostly the same... 

Another situation that makes things a bit tricky is if your main method implementation requires the use of properties or methods that would normally be out of scope once everything has been wrapped in a closure (i.e. private or protected members). For example, let's say our `tweet()` method logic makes use of a class called `OAuth` that's already a property of the `Twitter` class.

What you need to do is set up a local reference before you start the closure definition, making sure to use the `use` clause within the closure definition:

```php
class Twitter extends \lithium\core\StaticObject {

	public static function tweet($status, $options) {
		$auth = static::$_classes['oauth'];
		$defaults = ['responseFormat' => 'json'];
		$options += $defaults;

		$params = compact('status', 'options');

		return Filters::run(get_called_class(), __FUNCTION__, $params, function($params) use ($auth) {
			// Twitter connection and data transfer logic goes here...
			return $result;
		});
	}
}
```
