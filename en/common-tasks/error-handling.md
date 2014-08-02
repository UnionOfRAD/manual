# Exceptions and Error Handling

Error handling in Lithium is done using the core ErrorHandler class. `ErrorHandler` allows PHP errors and exceptions to be handled in a uniform way. Using `ErrorHandler`s configuration makes it possible to have broad but tightly controlled error handling across your application.

`ErrorHandler` configuration is done by creating a new error-specific bootstrap file that contains your `ErrorHandler` configuration and initialization. To illustrate how this is done, let's consider an imaginary (but common) scenario. Rather than tossing up error messages and stack traces to your users, it's better to create some sort of way to handle exceptions and render user-friendly error pages.

Let's start by creating a way to handle page not found-like errors. If a request can't be routed properly, the Lithium dispatcher will throw an exception to let you know that a controller or view can't be found. Though the example here will be specific to this case, it should provide a mental framework that will allow you to understand how to catch errors and exceptions, and handle them accordingly.

Start by creating a new bootstrap file in the application directory called `/config/bootstrap/error.php`:

```
use lithium\core\ErrorHandler;

$conditions = array('type' => 'lithium\action\DispatchException');

ErrorHandler::apply('lithium\action\Dispatcher::run', $conditions, function($exception, $params) {
	var_dump(compact('exception', 'params'));
	die();
});
```

This simple example shows how you can create a lambda that handles any `DispatchException`s being thrown in your entire application. The function you pass to `apply()` can be more involved, depending on what you want to do, however.

Here's a more complete example, showing how you'd actually render a template, and include logging:

```
use lithium\core\ErrorHandler;
use lithium\analysis\Logger;
use lithium\template\View;

Logger::config(array('error' => array('adapter' => 'File')));

$render = function($template, $content) {
	$view = new View(array(
		'paths' => array(
			'template' => '{:library}/views/{:controller}/{:template}.{:type}.php',
			'layout'   => '{:library}/views/layouts/{:layout}.{:type}.php',
		)
	));
	echo $view->render('all', compact('content'), compact('template') + array(
		'controller' => 'errors',
		'layout' => 'default',
		'type' => 'html'
	));
};
ErrorHandler::apply('lithium\action\Dispatcher::run', $conditions, function($exception, $params) {
	Logger::write('error', "Page Not Found...");
	$render('404', compact('exception', 'params'));
});
```

If you've got more than one type of exception you want to handle, just add more calls to `apply()` in your error bootstrap file.
