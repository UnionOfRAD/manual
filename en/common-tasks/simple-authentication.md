# Simple Authentication in li3

If you're doing much more than simple static content delivery, chances are you'll end up needing to protect access to certain resources and functionality your application provides. li3's authentication setup is simple and allows you to quickly create a framework for managing and protecting those resources.

## Data Setup

The default auth setup makes decisions based on information in your data store. The first thing you'll need to do is set up a model that handles user credentials. That model first needs a connection: set that up first in `config/bootstrap/connections.php`. If you're using MySQL as your data source, it should look something like this:

```php
use lithium\data\Connections;

Connections::add('default', array(
	'type'     => 'database',
	'adapter'  => 'MySql',
	'database' => 'mydatabase',
	'user'     => 'myusername',
	'password' => 'mypassword'
));
```

If you're running with Mongo, it'll look a bit different:

```php
Connections::add('default', array(
	'type'     => 'MongoDb',
	'database' => 'mydatabase',
));
```

Developers using MySQL will need a `users` table with at least the columns `id`, `username`, and `password`. Those using Mongo will need a collection in the database with a similar structure. You can customize the model and fields that `Auth` will use as we'll see later. Make sure to take a moment and set up your `Users` model as well in `models/Users.php`:

```php
namespace app\models;

class Users extends \lithium\data\Model {}
```

Once you've got that setup, your application more or less interacts with the data in the same way, regardless of the particular data source you're using.

## User Creation

Creating users that are compatible with the `Auth` setup is worth noting. While configurable, the default setup assumes a users data source with a hashed `password` field, and a `username` field. Please keep these defaults in mind as you create controller logic or view forms that interact with user data.

For convenience, we also recommend setting up a filter to automatically hash user passwords when new users are created. You can see an example filter in the __Model__ section of the [MVC auth setup instructions](simple-auth-user.md). You can continue to follow those instructions to create a controller and template that will allow you to generate users within your application.

## Bootstrapping Auth

Once the data handling is in place, li3 needs to know you intend to use authentication, and with which settings. As with most things, this is done in a specific bootstrap file.

li3's default application template ships with a file called `config/bootstrap/session.php`, which contains default session storage settings, as well as a commented-out default authentication configuration. To enable this configuration, first edit `config/bootstrap.php` to include (or uncomment) the line requiring the session bootstrap file:

```php
require __DIR__ . '/bootstrap/session.php';
```

Next, make sure your `Session` setup is using the PHP adapter, then create or uncomment the initial `Auth` configuration:

```php
use lithium\storage\Session;
use lithium\security\Auth;

Session::config(array(
	'default' => array('adapter' => 'Php')
));

Auth::config(array(
	'default' => array('adapter' => 'Form')
));
```

The `Session` setup is pretty straightforward, and the `Auth` configuration tells li3 which adapter we want to use (one suited for credentials submitted via web form), and details about the model involved and used to match incoming credentials against.

Note that the configuration information is housed in an array keyed `'default'`. `Auth` supports many different simultaneous configurations. Here we're only creating one, but you can add more (each using different adapters/configurations) as needed.

If you're editing the bootstrap files that ship with each new application, you'll notice that the configuration presented above is quite a bit simpler than the configuration provided. This is because the manual assumes you are following the default conventions, whereas the bootstrap files make it more explicit as to what configuration options are available.

## Authentication and Controller Actions

The first action you'll need to setup is the one that authenticates your users and adjusts the session to mark the user as identified. You could place this as you please, but it's generally accepted to create it in `SessionsController::add()`. The suggested approach is very simple:

```php
namespace app\controllers;

use lithium\security\Auth;

class SessionsController extends \lithium\action\Controller {

	public function add() {
		if ($this->request->data && Auth::check('default', $this->request)) {
			return $this->redirect('/');
		}
		// Handle failed authentication attempts
	}

	/* ... */
}
```

The meat is the part of the conditional that calls `Auth::check()` and hands it the name of our desired `Auth` configuration name (remember we named the whole config array `'default'`?), and information about the current request.

If the user has been successfully verified, the session is updated to mark the user as authenticated and the user is redirected to the root of the application. If there are problems with the authentication process the login view is rendered again.

As a reference, the web form that sends the credentials and is the content of the `add` view at `views/sessions/add.html.php` should contain something that looks like this:

```php
<?=$this->form->create(null); ?>
	<?=$this->form->field('username'); ?>
	<?=$this->form->field('password', array('type' => 'password')); ?>
	<?=$this->form->submit('Log in'); ?>
<?=$this->form->end(); ?>
```

## Protecting Resources

The setup for protecting resources is almost the same as it is for initially authenticating the user (though you'd want to redirect the user to the login action on error). Use `Auth::check()` in your controller actions to make sure that sections in your application are blocked from non-authenticated users. For example:

```php
namespace app\controllers;

use lithium\security\Auth;

class PostsController extends \lithium\action\Controller {

	public function add() {
		if (!Auth::check('default')) {
			return $this->redirect('Sessions::add');
		}

		/* ... */
	}
}
```

## Checking Out

Next, you'll want to create an action that clears an end-user's authentication session on your system. Do that by making a call to `Auth::clear()` in a controller action.

```php
namespace app\controllers;

use lithium\security\Auth;

class SessionsController extends \lithium\action\Controller {

	/* ... */

	public function delete() {
		Auth::clear('default');
		return $this->redirect('/');
	}
}
```

## Routing

Finally, in order to give users slightly friendlier URLs than `/sessions/add` and `/sessions/delete`, you can wire up some very simple custom routes in `config/routes.php`. Since these are very specific routes, you'll want to add them at or near the top of your routes file:

```php
Router::connect('/login', 'Sessions::add');
Router::connect('/logout', 'Sessions::delete');
```

## More Information

If you'd like to get under the hood and see how li3 handles password hashing, auth queries & session writes, etc., check out the API documentation for the following:

 - [The `Auth` class](http://li3.me/docs/lithium/security/Auth)
 - [The `Form` auth adapter](http://li3.me/docs/lithium/security/auth/adapter/Form)
 - [The `Password` class](http://li3.me/docs/lithium/security/Password)
