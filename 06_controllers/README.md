# Controllers

In an MVC application, the controller's job is to handle user input, and handle that request with a specified action. Though the controller action is where action logic is determined, models are often the real workhorses in the request cycle. Keep in mind as you read this guide to keep your controller action trim, with most of the actual functionality spread across models and other classes.

This guide is meant to introduce you to li3 controllers: their features, behaviors, and the best practices revolving around their usage.

## Controller Actions

li3 controllers reside inside the `/app/controllers` directory and extend the `lithium\action\Controller` core class. Let's start by creating a simple controller inside of an application. Controllers are often named after the objects they manage. This way the URL and model line up as well, and it's easy to know where certain bits of logic should live.

For example, let's create a new controller UsersController. Let's create a new file in `/app/controllers/UsersController.php` that looks like this:

```php
namespace app\controllers;

class UsersController extends \lithium\action\Controller {

	public function index() {}
}
```

Each _public_ function in a controller is considered by the li3 core to be a routable action. In fact, li3's default routing rules make these actions accessible via a browser immediately (in this case /users/index). 

The `index()` action is a special action: if no action name is specified in the URL, li3 will try to pull up the index action instead. For example, a visitor accessing http://example.com/users/ on your application will see the results of the `index()` action. All other controller actions (unless routed otherwise) are at least accessed by the default route.

For example, we can create a new controller action that would be accessible at `/users/view/`:

```php
namespace app\controllers;

class UsersController extends \lithium\action\Controller {

	public function index() {}

	public function view() {}
}
```

## Accessing Request Parameters

An important part of a controller's role in an application is processing incoming request data to show the correct response. In this section, we'll show a few examples that should help you to get data into your controller actions.

### GET Parameters

One of the most user-friendly ways to handle incoming data is through the URL. Information passed along with a GET request is handled a number of different ways, based on your preferences.

The easiest way to handle incoming GET data is by realizing that URL segments that follow the action name are mapped to controller action parameters. Here's a few examples:

```text
http://example.com/users/view/1  --> UsersController::view($userId);
http://example.com/posts/show/using+controllers/8384/  -->  PostsController::show($title, $postId);
```

GET information passed this way is also accessible via the incoming request object:

```text
http://example.com/users/view/1  --> $this->request->args[0]
```

While we'll always recommend using clear URL-based variables, it's important to mention that GET parameters passed as raw query string variables are also available as an attribute of the incoming request:

```text
http://example.com/users/view?userId=1  --> $this->request->query['userId']
```

### POSTed Data

POSTed data is also gathered from the request object. Form field data is found on the request object inside an array with keys named after the input elements generated on the referring page. For example, consider an HTML form that included these elements:

```html
<input type="text" name="title" value="How to Win Friends and Influence People" />
<input type="text" name="category" value="Self-Help" />
```

Accessing these values when submitted to a controller action is as easy as:

```php
$this->request->data['title'];
$this->request->data['category'];
```

## Request Flow Control

Occasionally a controller action will want to divert, re-route, or automatically configure the view layer based on an incoming request. There are a number of controller methods to help facilitate request flow handling.

### Redirection

The most basic type of flow control at the controller level is redirection. It's common to redirect a user to a new URL once an action has been performed. This type of control is done through the controller's `redirect()` method. Here's an example of a controller action that redirects the request:

```php
public function register() {
	// Validate and save user data POSTed to the 
	// controller action found in $this->request->data...

	$this->redirect('Users::welcome');
}
```

The URL specified can be relative to the application or point to an outside resource. The `redirect()` function also features a second `$options` parameter that also allows you to set HTTP status headers, and make decisions about whether or not to `exit()` after a redirect. Be sure to check the API for `lithium\action\Controller::redirect()` for more details.

### Exceptions and Error Handling

Error handling in li3 is done using the core ErrorHandler class. `ErrorHandler` allows PHP errors and exceptions to be handled in a uniform way. Using `ErrorHandler`s configuration makes it possible to have broad but tightly controlled error handling across your application.

`ErrorHandler` configuration is done by creating a new error-specific bootstrap file that contains your `ErrorHandler` configuration and initialization. To illustrate how this is done, let's consider an imaginary (but common) scenario. Rather than tossing up error messages and stack traces to your users, it's better to create some sort of way to handle exceptions and render user-friendly error pages.

Let's start by creating a way to handle page not found-like errors. If a request can't be routed properly, the li3 dispatcher will throw an exception to let you know that a controller or view can't be found. Though the example here will be specific to this case, it should provide a mental framework that will allow you to understand how to catch errors and exceptions, and handle them accordingly.

Start by creating a new bootstrap file in /app/config/bootstrap/error.php:

```php
use lithium\core\ErrorHandler;

$conditions = ['type' => 'lithium\action\DispatchException'];

ErrorHandler::apply('lithium\action\Dispatcher::run', $conditions, function($exception, $params) {
	var_dump(compact('exception', 'params'));
	die();
});
```

This simple example shows how you can create a lambda that handles any `DispatchException`s being thrown in your entire application. The function you pass to apply() can be more involved, depending on what you want to do, however.

Here's a more complete example, showing how you'd actually render a template, and include logging:

```php
use lithium\core\ErrorHandler;
use lithium\analysis\Logger;
use lithium\template\View;

Logger::config(['error' => ['adapter' => 'File']]);

$render = function($template, $content) {
	$view = new View([
		'paths' => [
			'template' => '{:library}/views/{:controller}/{:template}.{:type}.php',
			'layout'   => '{:library}/views/layouts/{:layout}.{:type}.php',
		]
	]);
	echo $view->render('all', compact('content'), compact('template') + [
		'controller' => 'errors',
		'layout' => 'default',
		'type' => 'html'
	]);
};

ErrorHandler::apply('lithium\action\Dispatcher::run', [], function($exception, $params) use ($render) {
	Logger::write('error', "Page Not Found...");
	$render('404', compact('exception', 'params'));
});
```

If you've got more than one type of exception you want to handle, just add more calls to `apply()` in your error bootstrap file.

### Render Types and Detection

Although a typical request to a li3 application receives an HTML response, the framework is built to be extremely flexible in handling and serving different types of content. This functionality is especially important in applications that have many different components or endpoints. If your app also feeds data to a Flash object (AMF/XML) and a mobile phone (XML/JSON), responding to requests in different ways with the same underlying logic can be a huge time saver.

The flow for handling a given type of a response works something like the following:

 1. A request is sent to the application, containing some sort of indicator of the request type. li3's default routing allows for simple extension detection, for example. 
 2. As li3 bootstraps, a media type and handler is registered with the `\net\http\Media` class.
 3. The application detects the request type and sets the response type.
 4. Once a controller is ready to render the data, the registered handler receives the data and renders the output.

#### Detecting and Setting Types

The easiest way to set a type is by declaring it as part of the route. One of li3's default routes already does this for you:

```php
Router::connect('/{:controller}/{:action}/{:id:[0-9]+}.{:type}', ['id' => null]);
```

In effect, this forces a request to `/controller/action/7345.json` to be rendered by the JSON media handler currently registered. You can use this pattern to expand your routes to apply type-matching to a wider array of requests:

```php
// http://example.com/controller/action.xml
Router::connect('/{:controller}/{:action}.{:type}');
```

You can also statically define the type for a route by adding the 'type' key to the route definition:

```php
Router::connect('/latest/feed', [
	'Posts::index',
	'type' => 'xml'
]);
```

If you'd rather use other information to convey the request type to li3 (headers, GET variables, etc.) you can gather that information then set `$this->_render['type']` in the controller action.

Manual type rendering can also be done by handing the type's name to the render function:

```php
$this->render(['csv' => Post::find('all')]);
```

#### Handler Registration

As mentioned earlier, type handlers are registered by the `\net\http\Media` class. This is usually done in the `/config/bootstrap/media.php` bootstrap file. Be sure to uncomment the corresponding line in the main bootstrap file to enable this functionality.

Register your type by passing the relevant information to `Media::type()` inside the bootstrap. Here's what the general pattern looks like:

```php
Media::type('typeName', 'contentType', [$options]);
```

To give you an idea of how this process is completed, let's register a new handler for the BSON data type. If you're curious, BSON is a binary, serialized form of JSON used by MongoDB. Start by declaring a new media type in `/config/boostrap/media.php` and uncommenting the `media.php` line in your main bootstrap file:

```php
Media::type('bson', 'application/bson', []);
```

This gets us pretty far. If you make a request with a .bson extension that matches a configured route (one with {:type} in it), li3 will already hunt for a `.bson.php` template in the controller's template directory. You can continue to customize li3's behavior by utilizing the `$options` array you supply to `Media::type()`. After checking the API docs for `Media::type()`, we realize we can utilize a few options to make sure our response isn't housed in an HTML layout and use some functions we've defined for encoding and decoding the data:

```php
Media::type('bson', 'application/bson', [
	'layout' => false,
	'encode' => 'bson_encode',
	'decode' => 'bson_decode'
]);
```

Try the request again, and you'll get your BSON-encoded data, minus the HTML layout. Note that the bson_* functions used in this particular example are part of the PECL MongoDB extension. Don't worry if you don't have it installed: the main point is to realize that you can tell `Media` exactly what function to use to render the data (including using closures).

For more ideas on configuring media types, see the documentation for `Media::type()`.

## Using Models and Core Libraries

While an entire guide is devoted to covering model usage, it's important to see how they're used inside the controller layer. Using models inside li3 controllers is simple. Let's start with a bare controller as an example:

```php
namespace app\controllers;

class ClientsController extends \lithium\action\Controller {

	public function index() {}
}
```

While not required, it's helpful to name controllers after the models they primarily use, at least for organizational purposes. To start using your model inside this controller, you'll need to let PHP know you intend to use the model class inside this controller:

```php
namespace app\controllers;

use app\models\Client;

class ClientsController extends \lithium\action\Controller {

	public function index() {}
}
```

Since most model access is done statically, just access the methods you need in your controller actions directly:

```php
namespace app\controllers;

use app\models\Client;

class ClientsController extends \lithium\action\Controller {
	
	public function index() {
		$clients = Client::find('all');
		return compact('clients');
	}

	public function details($clientId) {
		$client = Client::find('first', [
			'conditions' => [
				'id' => $clientId,
			]
		]);

		return compact('client');
	}
}
```

Using li3's core libraries inside the controller layer is similar, although instantiating a class is sometimes necessary. Consider the following example that uses the `Service` class:

```php
namespace app\controllers;

use SimpleXmlElement;
use lithium\net\http\Service;

class ClientsController extends \lithium\action\Controller {

	public function index() {
		$service = new Service(['host' => 'api.flickr.com']);
		$response = new SimpleXmlElement($service->get('/services/feeds/photos_public.gne'));
		return compact('response');
	}
}
```

This is done by declaring the usage of the class by specifying it's fully namespaced path, and later instantiating it inside the action logic.

Also note that core PHP classes (`SimpleXmlElement`, in this case) are imported with the `use` statement as well, since they exist in the root namespace.

## Moving Data to the View

Once your controller action has fetched and processed the data it needs, it's time to push the data to the view layer. There are two main ways to accomplish this: by using the `set()` method of the `Controller` class, or by simply returning an associative array as the result of a controller action method.

First, the `set()` method is used to send an associative array to the view. Consider this example controller action method:

```php
public function index() {
	$data = SomeModel::find('all');
	$this->processData($data);
	$this->set(['importantData' => $data]);
}
```

This populates the index view for this controller with a variable named `$importantData` with the same contents as `$data`. This same logic could be written a bit more elegantly:

```php
public function index() {
	$data = SomeModel::find('all');
	$this->processData($data);
	$this->set(compact('data'));
}
```

One difference to note here is that the view's variable is now named `$data`, just as it was in the controller.

The `set()` method is especially useful when you've got bits of data you want to hand to the view in pieces, or conditionally, as it can be called at any point inside of a controller action method:

```php
public function index() {
	$data = SomeModel::find('all');
	$this->processData($data);
	$this->set(['importantData' => $data]);

	$moreData = SomeService::get();
	$this->processData($moreData);

	if ($someCondition) {
		$this->set($moreData);
	}

	//...
}
```

Conversely, sometimes all your data is ready by the end of the method's logic, and you can just return the associative array to hand it to the view:

```php
public function index() {
	$data = SomeModel::find('all');
	$this->processData($data);

	$moreData = SomeService::get();
	$this->processData($moreData);

	return compact('data', 'moreData');
}
```

## Filtering Controller Logic

Filtering controller logic can be a bit tricky to the new li3 user, due to the flexible and elegant way the dispatch cycle executes.

When a request is sent to an application, the Dispatcher first uses the Router to determine which controller to load. Once a target controller has been identified, a new controller object is created and invoked.

This last invokation step is performed inside of the dispatcher's `_callable()` method. Because of this, filters that wish to inject logic before or after controller actions is often done against `_callable()`. Doing so allows you access to parameters that are normally available to the controller, as well as the right controller instance, allowing you to call its methods.

 _Note:_ You might have seen some filters run against the dispatcher's `run()` method instead. In many cases, this should work just fine. The difference in using `_callable()` is that with the latter, you'll end up with access to the active controller instance itself, along with its parameters.

The `g11n` filters that come with the standard distribution form an illustrative example. Consider this slightly modified (for simplicity) version of that same filter:

```php
use lithium\aop\Filters;
use lithium\action\Dispatcher;

Filters::apply(Dispatcher::class, '_callable', function($params, $next) {
	$request = $params['request'];
	$controller = $next($params);

	if (!$request->locale) {
		$request->params['locale'] = Locale::preferred($request);
	}
	Environment::set(Environment::get(), ['locale' => $request->locale]);
	return $controller;
});
```

<div class="note note-info">This example uses new-style filters, available with 1.1.</div>

Here you can see that you've got access to the request parameters and the controller instance itself in the first two lines of the `Filters::apply()` call. In this case, the g11n framework is inspecting the request and setting locale settings accordingly, but when creating your own filter, you'd have access to the same data.

An important case to consider that's also covered in the g11n filters is remembering to also filter the frameworks other dispatcher: the console action dispatcher. In other words, if you truly want to effect every action in an application, you might consider also filtering the `lithium\console\Dispatcher` as well as the `lithium\action\Dispatcher`.

In this case, you'll want to use the special use/as syntax:

```php
use lithium\aop\Filters;
use lithium\action\Dispatcher as ActionDispatcher;
use lithium\console\Dispatcher as ConsoleDispatcher;

Filters::apply(ActionDispatcher::class /*, ... */);
Filters::apply(ConsoleDispatcher::class /*, ... */);
```
