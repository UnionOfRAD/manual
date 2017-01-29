# Creating a user in M, V, C

First, create a database with a `users` table or collection, with at least a primary key (usually `id` or `_id`) and the fields `username` and `password` (these could of course be renamed, but that requires more configuration).

## The Model

Create the model in `models/Users.php`:

```php
namespace app\models;

class Users extends \lithium\data\Model {}
```

Then, create a model filter in a new bootstrap file in `config/bootstrap/` called `user.php`. This file will automatically hash user passwords before the accounts are created. The `Password` class will automatically use the most secure hashing method available on your system:

```php
use app\models\Users;
use lithium\aop\Filters;
use lithium\security\Password;

Filters::apply(Users::class, 'save', function($params, $next) {
	if ($params['data']) {
		$params['entity']->set($params['data']);
		$params['data'] = [];
	}
	if (!$params['entity']->exists()) {
		$params['entity']->password = Password::hash($params['entity']->password);
	}
	return $next($params);
});
```

<div class="note note-info">This example uses new-style filters, available with 1.1.</div>

Now add the following line to config/bootstrap.php to include your new user.php bootstrap file.

```php
require __DIR__ . '/bootstrap/user.php';
```

## The Controller

Create this file in `controllers/UsersController`:

```php
namespace app\controllers;

use lithium\security\Auth;
use app\models\Users;

class UsersController extends \lithium\action\Controller {

	public function index() {
		$users = Users::all();
		return compact('users');
	}

	public function add() {
		$user = Users::create($this->request->data);

		if (($this->request->data) && $user->save()) {
			return $this->redirect('Users::index');
		}
		return compact('user');
	}
}
```

## The views

Then create the templates.

`views/users/add.html.php`:

```html
<h2>Add user</h2>
<?= $this->form->create($user) ?>
	<?= $this->form->field('username') ?>
	<?= $this->form->field('password', ['type' => 'password']) ?>
	<?= $this->form->submit('Create me') ?>
<?= $this->form->end() ?>
```

`views/users/index.html.php`:

```html
<h2>Users</h2>

<ul>
	<?php foreach ($users as $user) { ?>
		<li><?= $user->username ?></li>
	<?php } ?>
</ul>
```
