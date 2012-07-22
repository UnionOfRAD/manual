# The Lithium Filters Guide

This guide is meant as an introductory course in creating filters in Lithium applications. Filters are essentially an efficient way of introducing event-driven communication between classes in your application. They allow you to inject bits of logic in the middle of main-line program flow while at the same time keeping the API clean, avoiding tight class coupling and some sort of centralized publish/subscribe system.

What would you need a filter for? Here are some quick examples:

* A 'before filter' that marks the 'created' column in your database with the current date
* Intercepting certain requests to deliver raw file contents rather than an HTML response
* Wrapping profiling measurements around each dispatched request
* Checking each request to make sure the user has been authenticated
* Logging various events in your application
* Creating a read-through cache filter that encapsulates the process of checking whether a result has been cached, serving a cached result, and writing results to a cache backend

By the end of this guide, you should feel comfortable in your understanding of the theory and best practices behind filters, as well as identifying filterable logic in Lithium and creating your own filterable methods.

## Aspect-Oriented Programming

Lithium's filter system is inspired by similar concepts in Aspect-Oriented Programming, or AOP. In AOP, an aspect refers to a bit of logic that is scattered throughout an application. Usually, it's a piece of logic that is used in a number of different points in the application, often in completely unrelated systems.

There are a few examples of aspects that you're probably already familiar with. Logging and authentication are two great ones. Both are used in many different parts of an application, and maintaining something so widespread can get tricky unless you're organized. Not only that, those bits of logic aren't really part of the code that makes up your business logic. In other words, they're separate _concerns_.

## Filter Structure

Long story short, a filterable method is created by encapsulating the main logic for a method inside a closure. That closure is then passed to `Object::_filter()` (or `StaticObject::_filter()` for static objects) to allow for chain management.

Applying a custom filter to a filterable method is done by calling the `applyFilter()` method of the object. The first argument specifies the name of the method to filter, and the second argument is the closure that defines what you want the filter to do.

The following sections will illustrate a few examples that should help you understand how both to apply and create filters.

## Applying Filters: Authentication

Since it's likely to be more common, let's tackle applying filters first. The location of the filter application might depend some on what it is you're filtering, but bootstrap files are often a good choice.

Let's imagine for a moment that we're aiming to add a filter to a Lithium application that checks for authenticated users. Let's start with a basic filter setup, and place it in `app/bootstrap/session.php` (if you look at `session.php`, you'll notice there's already some configuration for defining session storage and authentication). The thinking here is that we want to inject a bit of authentication verification logic as the dispatcher receives each request. Since `Dispatcher::run()` is filterable, we'll apply the filter to it:

{{{
use lithium\action\Dispatcher;

Dispatcher::applyFilter('run', function($self, $params, $chain) {
	return $chain->next($self, $params, $chain);
});
}}}

This is the basic structure for applying a filter: call the apply `applyFilter()` method on the class or object in question, hand it the name of the method you'd like to filter, then a closure that implements your filter logic. Let's talk about the parameters involved in the closure:

* `$self`: If the filter is applied on an object instance, then `$self` will be that instance. If applied to a static class, `$self` is a string containing the fully-namespaced class name.

* `$params`: Contains an associative array of the parameters that are passed into the method. You can modify or inspect these parameters before allowing the method to continue.

* `$chain`: Finally, `$chain` is an instance of the `lithium\util\collection\Filters` class, and it contains the list of filters in line to be executed. The last item in `$chain` is the original method implementation itself (we'll look at this more later).

The first parameter, `$self`, is present to allow you to use the other methods or properties on the object you're filtering.

Next, `$params` gives you access to any parameters that original method implementation had access to. The bulk of your logic will often be in accessing or modifying what's in those parameters.

The last parameter is important. You can see above that right now the closure returns the results of the logic next in line in the filter chain. Unless you want to short-circuit filter execution (which includes the base logic of the method you're filtering), you'll want to include this somewhere in your filter logic. The position this call has in your filter also might be important. For example, if you're creating a filter you want to have happen __after__ a certain method, your custom filter logic should follow the `next()` call rather than proceed it:

{{{

SomeClass::applyFilter('methodName', function($self, $params, $chain) {
	$result = $chain->next($self, $params, $chain);

	// Custom logic goes here.
});

}}}

Alright: let's use these parameters to our advantage in creating an authentication setup. If we look at the parameters for `Dispatcher::_callable()` we can see that accepts three: the current `Request` object (an instance of `lithium\action\Request`), the parameters returned from routing the request, and an options array.

When designing filters, it's important to have a clear understanding of the method you're hooking into, and where it fits in the framework's lifecycle. The `_callable()` method is invoked immediately after the application's routes have been processed, and routing parameters have been returned. This method's responsibility is to take those routing parameters and convert them to a callable object (i.e. a controller) that can process the request and turn it into a response.

For this filter, we'll intercept the method after it finds a controller object (assuming it finds one), but before it returns it to `Dispatcher::run()`:

{{{
use lithium\action\Dispatcher;
use lithium\action\Response;
use lithium\security\Auth;

Dispatcher::applyFilter('_callable', function($self, $params, $chain) {
	$ctrl    = $chain->next($self, $params, $chain);
	$request = isset($params['request']) ? $params['request'] : null;
	$action  = $params['params']['action'];

	if (Auth::check('default')) {
		return $ctrl;
	}
	if (isset($ctrl->publicActions) && in_array($action, $ctrl->publicActions)) {
		return $ctrl;
	}
	return function() use ($request) {
		return new Response(compact('request') + array('location' => 'Sessions::add'));
	};
});
}}}

This particular listing assumes you've set up a configuration with the `Auth` class called `'default'`. Check out the authentication chapter for more on this. Taking it from the top, we can see that the filter first continues executing the chain in order to get the `Controller` object (`$ctrl`) to work with, and does all of its processing _after_ the actual method executes.

Next, it uses the `Auth` class to see if a user is logged in, and if so, allows the application to continue normally. Then, it checks to see if the controller has a `$publicActions` property defined, and if so, checks the array of whitelisted actions to see if the requested one matches, and again, allows processing to continue if so.

Finally, if the user is not logged in, and the requested action is not public, the user is redirected to the login page. Note that, as opposed to returning a `Response` object directly, as would be appropriate when filtering other `Dispatcher` methods like `run()` or `_call()`, here we are returning a closure.

It is important when filtering a method that we honor the method's _contract_. That is, when filtering a method, if we intercept and modify its parameters, or if we bypass it and substitute the return value, we must still keep the values within the acceptable range of what the method accepts or produces. In the case of the `_callable()` method, it is expected to produce a callable object. The `Controller` uses PHP's magic `__invoke()` method to achieve this. Therefore, we can interchange controller objects with closures, and vice versa.

Implementing the controller for this example might look something like this:

{{{
namespace app\controllers;

use lithium\security\Auth;

class SessionsController extends \lithium\action\Controller {

	public $publicActions = array('add');

	public function add() {
		if ($this->request->data) {
			$success = Auth::check('default', $this->request);
			// ...
		}
		// ...
	}
}
}}}

The rest of the code necessary to implement user authentication is covered in another chapter, but take special note of the `$publicActions` property in the controller. Without this, the `add()` action would be treated as private, resulting in an infinite redirect loop.

Although authentication is a very deep subject, through this example you should be able to see how, whatever your project's rules and constraints are, integrating into your application should be straightforward, and most importantly, unobtrusive.

## Applying Filters: Logging

One more example is common enough to cover: logging. When building a large application, it's handy to log SQL queries made against your data store. It helps in debugging and performance optimization. Let's cover building a quick filter around your data connection logic in order to log SQL statements to a file.

The approach here requires a bit of understanding how Lithium's data layer works. In the setup of your application, you've probably created new database (or other datasource) connections in `app/config/bootstrap/connections.php`. Here's what a simple connection to a MySQL database might look like:

{{{
Connections::add('default', array(
	'type' => 'database',
	'adapter' => 'MySql',
	'host' => 'localhost',
	'login' => 'myusername',
	'password' => 'mypassword',
	'database' => 'app_name'
));
}}}

What we need to do is get a reference to the instance of the actual connection object define here so we can filter its methods. In this case, we'll use `Connections::get()` to get a reference to the actual adapter handling this connection. Since we've specified 'MySQL' as the adapter type, we'll get an instance of `lithium\data\source\database\adapter\MySql` when we call `Connections::get('default')`.

Looking at the API shows us that there's an `_execute()` method we can filter. It's only parameter is the SQL being executed, and that's exactly what we'll need for our filter logic. Here's one way this could look:

{{{
use lithium\analysis\Logger;
use lithium\data\Connections;

// Set up the logger configuration to use the file adapter.
Logger::config(array(
	'default' => array('adapter' => 'File')
));

// Filter the database adapter returned from the Connections object.
Connections::get('default')->applyFilter('_execute', function($self, $params, $chain) {
	// Hand the SQL in the params headed to _execute() to the logger:
	Logger::debug(date("D M j G:i:s") . " " . $params['sql']);

	// Always make sure to keep the filter chain going.
	return $chain->next($self, $params, $chain);
});
}}}

## Applying Filters Lazily

One issue that may arise when applying filters to static classes inside of bootstrap logic is triggering the autoloading of those classes. This consumes memory, as well as invoking any other processes which may be triggered when a class is loaded.

For this reason, it is sometimes preferrable to apply filters to a class without direct invocation. The `Filters` class, which powers the filter system, includes an `apply` method that enables this.

Using the fully-qualified name of the class to which you want to apply the filter, you may invoke `Filters::apply()`, which will store the filter until the method to which it is applied first executes:

{{{
use lithium\util\collection\Filters;

Filters::apply('blog\models\Posts', 'find', function($self, $params, $chain) {
	// Filter logic is then written the same as it would be for non-lazy filters
});
}}}

## Creating Filter-able Logic

If you're planning on creating and distributing your own code (hopefully via the [Lithium Laboratory](http://lab.lithify.me/), among other things) you might consider writing parts of your API to be filter-able. This allows other developers to take advantage of the powerful Lithium filter system, while at the same time helping you avoid writing extra callback methods or configuration options in your code.

Let's imagine for a bit that you're creating a social media integration extension for Lithium. The library will connect to popular social networking sites to post or gather information. Since you're offering this to other developers to use in their applications, it'd be nice to enable filtering on some of the logic.

For example, let's add a filter to the method that posts content to Twitter. It's conceivable that developers might want to filter that logic to shorten URLs using different services, or automatically include hashtags at the end of the tweet.

First, start with the basics:

{{{
class Twitter {

	public function tweet($status, $options) {

		$defaults = array('responseFormat' => 'json');
		$options += $defaults;

		// Twitter connection and data transfer logic goes here...
		return $result;
	}
}
}}}

Here we've got a basic method implementation. Of note is the `$options` parameter: the Lithium API is a big fan of keeping parameter counts small using options arrays. Coding to that same standard helps developers more quickly understand how to put things together. Here you can see a bit of prep work happen before the main logic of the method, mostly just in merging some default values into the supplied options.

Once we've got the core logic in, adding the ability to filter it is relatively painless. What we end up doing is wrapping up the core implementation in a closure that we hand to `$this->_filter()`, and returning the result of that call. Here's what it looks like:

{{{
class Twitter extends \lithium\core\Object {

	public function tweet($status, $options) {

		$defaults = array('responseFormat' => 'json');
		$options += $defaults;

		$params = compact('status', 'options');

		return $this->_filter(__METHOD__, $params, function($self, $params) {
			// Twitter connection and data transfer logic goes here...
			return $result;
		});
	}
}
}}}

The `filter()` method takes three arguments. The first is the name of the method, the second is an array of parameters the original method implementation takes. The last parameter is a closure containing the original method implementation. That's it! If you're doing something pretty simple - that's all you need to do. Note that in order to call `$this->_filter()`, the class must extend `lithium\core\Object` (or a subclass thereof).

There are a few situations that make things a bit different. One example is if your method is accessed statically. Let's adjust our `tweet()` example to illustrate (note that the parent class also changes):

{{{
class Twitter extends \lithium\core\StaticObject {

	public static function tweet($status, $options) {

		$defaults = array('responseFormat' => 'json');
		$options += $defaults;

		$params = compact('status', 'options');

		return static::_filter(__FUNCTION__, $params, function($self, $params) {
			// Twitter connection and data transfer logic goes here...
			return $result;
		});
	}
}
}}}

Mostly the same... just call `static::_filter()` instead of `$this->_filter()`.

Another situation that makes things a bit tricky is if your main method implementation requires the use of properties or methods that would normally be out of scope once everything has been wrapped in a closure (i.e. private or protected members). For example, let's say our `tweet()` method logic makes use of a class called `OAuth` that's already a property of the `Twitter` class.

What you need to do is set up a local reference before you start the closure definition, making sure to use the `use` clause within the closure definition:

{{{
class Twitter extends \lithium\core\StaticObject {

	public static function tweet($status, $options) {
		$auth = static::$_classes['oauth'];
		$defaults = array('responseFormat' => 'json');
		$options += $defaults;

		$params = compact('status', 'options');

		return static::_filter(__FUNCTION__, $params, function($self, $params) use ($auth) {
			// Twitter connection and data transfer logic goes here...
			return $result;
		});
	}
}
}}}