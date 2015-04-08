# Validation

This section focuses on validation in the model layer, as well as covering the core utility class `Validator`.

Part of a model's responsibility in an MVC application is to be the gatekeeper for the data it handles. While a great deal of a model's functionality deals with fetching and relating data, validation is an important part in describing the business logic and keeping your data clean and secure.

## Basic Validation

Validation is done by defining a special property on the model object called `$validates`. When calling `validates()` on an entity the framework inspects this property and validates data to be saved against this set of rules. 

Because it's likely the most common case, we'll start with a simple model data validation example based around an HTML form. Let's consider an application that handles user registration. We'll define a `Users` model and create some simple validation logic in order to make sure submitted user information is complete and in the correct form.

### Defining Validation Rules

Let's create a simple `Users` model, and begin with the definition of a few simple validation rules.

```php
namespace app\models;

class Users extends lithium\data\Model {

	public $validates = array(
		'name' => array(
			array(
				'notEmpty',
				'required' => true,
				'message' => 'Please supply a name.'
			)
		),
		'password' => array(
			array(
				'notEmpty',
				'required' => true,
				'message' => 'Please supply a password.'
			)
		)
	);
}
```

<div class="note note-info">
Note that these are application-level validation rules, and do not interact with any rules or constraints defined in your data source. If such constraints fail, an exception will be thrown by the database layer. Validation checks run only against the rules defined in application code. 
</div>

This initial example shows how rules are defined. First, each set of rules is keyed by the model field name. Each field is then assigned an array of rules, where the first value is the name of the rule, and the subsequent keys define additional information about each rule.

In addition to the rule name, there are a number of special keys you can use to modify a rule:

 - `message`: The error message displayed if this rule fails.
 - `required` (boolean): Specifies that data for this field _must_ be submitted in order to validate. Defaults to `true`.
 - `skipEmpty` (boolean): Causes the rule to be skipped if the value is null or empty. Defaults to `false`.
 - `format`: The name of the rule format required to validate the data, or `any` or `all`.

You may also declare multiple rules per field. Here, we add an additional rule to the name field in order to ensure names are alphanumeric.

```php
namespace app\models;

class Users extends lithium\data\Model {

	public $validates = array(
		'name' => array(
			array(
				'notEmpty',
				'message' => 'Please supply a name.'
			),
			array(
				'alphaNumeric',
				'message' => 'A name may only contain letters and numbers.'
			)
		),
		'password' => array(
			array(
				'notEmpty',
				'message' => 'Please supply a password.'
			)
		)
	);
}
```

<div class="note note-hint">
	For a complete list of built-in validation rules, see the <a href="http://li3.me/docs/lithium/util/Validator"><code>Validator</code> API</a>.
</div>

### Form Validation

Here's a simple form we might use to collect user data. This would be contained in `app/views/users/add.html.php`. 

```
<?= $this->form->create($user); ?>
	<?= $this->form->field('name'); ?>
	<?= $this->form->field('password'); ?>
	<?= $this->form->submit('save user'); ?>
<?= $this->form->end(); ?>
```

By using the `Form` helper and binding our user entity to it, we gain the additional benefit that later validation errors are handled and - should some exist - displayed automatically for us. 

### Handling Validation in the Controller

Once that data is submitted through the form to the controller, we handle it in our controller action. 

```php
namespace app\controllers;

use app\models\Users;

class UsersController extends lithium\action\Controller {

	public function add() {
		if ($this->request->data) {
			$user = Users::create($this->request->data);

			if ($user->save()) {
				$this->redirect('/users/home');
			}
		}
	}
}
```

If there's submitted data to this action, we create a new user with it. If a save is successful, we redirect to a landing page. The **implicit logic** here is that validation rules are checked, and `save()` returns a `false` if any rules fail.

Since the action's view contains the HTML form we just built using the `Form` helper, if there are any validation problems, the HTML form is rendered again, this time with the errors displayed inline.

## Advanced Validation

### Explicit Validation

As shown in our example above `save()` will implictly validate the data prior saving. If you'd like
to validate data explictly you'd use the entities `validate()` method. That method takes 2 parameters. 

The `$entity` parameter specifies the model entity to validate, which is typically either a `Record` or `Document` object. In the  example below, the `$entity` parameter is equal to the `$post` object instance.

The ``$options` parameter has the following settable elements:

* `'rules'` _array_: If specified, this array will _replace_ the default validation rules defined in `$validates`.
* `'events'` _mixed_: A string or array defining one or more validation _events_. Events are different contexts in which data events can occur, and correspond to the optional `'on'` key in validation rules. For example, by default, `'events'` is set to either `'create'` or `'update'`, depending on whether `$entity` already exists. Then, individual rules can specify `'on' => 'create'` or `'on' => 'update'` to only be applied at certain times. Using this parameter, you can set up custom events in your rules as well, such as `'on' => 'login'`. Note that when defining validation rules, the `'on'` key can also be an array of multiple events.

```
$post = Posts::create($data);

if ($success = $post->validates()) {
	// ...
	$post->save(array('validate' => false));
}
```

This method uses the `Validator` class to perform data validation. An array representation of the entity object to be tested is passed to the `check()` method, along with the model's validation rules. Any rules defined in the `Validator` class can be used to validate fields.

<div class="note note-hint">
	When validation fails any validation errors get attached to the entity.
</div>


### Accessing Validation Errors

If you need to know about model validation problems before the application renders the view, you can inspect errors on the `Entity` object you're working with. Get the last errors by calling `errors()`:

```php
// ...
	public function add() {
		if ($this->request->data) {
			$user = Users::create($this->request->data);
			$user->save();
			$errors = $user->errors();

			// Redirect or perform other logic here
			// based on contents of $errors.
		}
	}
// ...
```

### Invalidating Manually

You can also manually invalidate model fields by calling `errors()` and supplying key/value pairs to denote problems. While you'll usually want to create custom rules (covered next), this is sometimes a helpful trick to use in model filters, etc.

```php
$user->errors('name', 'This name is too funny.');
```

### Creating Custom Rules

While the validator features a number of handy rules, you'll inevitably want to create your own validation rules. This done (at runtime) by calling `Validator::add()` to specify new rule logic.

The simplest form of rule addition is by Regular Expression:

```php
Validator::add('zeroToNine', '/^[0-9]$/');
```

If you need more than pattern recognition, you can also supply rules as anonymous functions:

```php
Validator::add('nameTaken' function($value) {
	$result = Users::findByName($value);
	return count($result) > 0;
});
```

Once a rule as been defined, it can be referenced by name in a model `$validates` property or direct use of the `Validator` class magic methods:

```php
Validator::add('zeroToNine', '/^[0-9]$/');
Validator::isZeroToNine('7');
```

### Named Validation Rules

Validation rules inside the model can be optionally defined as _named_ rules. This changes the format
of the data `Model::errors()` returns and allows you to determine which exact rule failed.

```php
namespace app\models;

class Users extends lithium\data\Model {

	public $validates = array(
		'name' => array(
			'foo' => array(
				'notEmpty'
				'message' => 'Must not be empty.'
				// ...
			),
			'bar' => array(
				// ...
			)
		)
	);
}
```

```php
$user->validate();
$user->errors(); // Returns:
// array(
//     'name' => array(
//         'foo' => 'Must not be empty'
//     )
// )
```

### Providing Custom Validation Messages in the Template

In order to make use of this feature, you must use _named validation rules_. 

This allows easier translation of messages and customization in case there is no
control over the model (i.e. developing a "theme" for a customer without touching
the data layer).

Also some might prefer to provide messages in the template, as one could argue
they are part of the presenation layer.

```php
$this->form->field('name', array(
	'error' => array(
		'foo' => 'Please please do not leave empty :)'	
	)			
));
```

Once a `'default'` key is set for the custom messages, it'll be used for any unmatched
validation errors.

```php
$this->form->field('name', array(
	'error' => array(
		'default' => 'Something is wrong in this field.'
		'foo' => 'Please please do not leave empty :)'	
	)			
));
```

<div class="note note-info">
	When there's no default message given, messages defined in the model
	may be rendered.
</div>
