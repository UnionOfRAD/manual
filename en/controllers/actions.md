# Actions
li3 controllers reside inside the application's `/controllers` directory and extend the `lithium\action\Controller` core class. Let's start by creating a simple controller inside of an application. Controllers are often named after the objects they manage. This way the URL and model line up as well, and it's easy to know where certain bits of logic should live.

For example, let's create a new controller UsersController. Let's create a new file in `/controllers/UsersController.php` that looks like this:

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
