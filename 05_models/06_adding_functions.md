# Adding Functionality

This section outlines a few of the ways you can extend model functionality as you build your application. As you think about different pieces of functionality to add, please keep in mind that there are two main ways to add functionality to your models:

 1. Static methods that apply to the Model
 2. Instance methods that apply to each Entity (Document or Record)

## Static vs Instance Methods

As a rule of thumb, all static methods work with a collection of objects on the database, while non-static methods are tied to one single document/record. Consider the following example:

```
$post = Posts::first(['conditions' => ['author' => 'foo']]);
$post->title = 'Hello World';
$post->save();
```

The `first()` method is static because it iterates over a bunch of entities in your database and returns the first entry where the `author` equals `foobar`. The result you get is an instance of a `Post` so all subsequent methods on it are _non-static_. The second line in the example sets the `title` and the third line saves it back to the database. This concept feels natural and also has the benefit of instantly knowing on what kind of dataset you're operating.

If you remember this simple rule, you'll understand how the framework reacts to new functions placed in models—and you'll be able to use them more effectively.

## Static Data Access

One simple example is if you have a bit of data that is specific to a model's domain, and you want to make that data available to controllers using those models.

```php
namespace app\models;

class Users extends \lithium\data\Model {

	protected static $_roles = ['admin', 'friend', 'stranger'];

	public static function roles() {
		return static::$_roles;
	}
}
```

Because this method is accessed statically, it behaves as you'd expect:

```php
if (!in_array('admin', Users::roles())) {
	return false;
}
```

## Model Instance Methods

It's often useful to add a method to a model so that you can easily transform data once you've got a model instance (of type `Entity`, either `Document` or `Record`) in your controllers and views. In this case, you'll need to create a method that accepts the entity itself as the first argument, then any addition parameters (if any). Here's a simple use case of creating a method on a `Users` model that formats a full name based on first and last names:

```php
namespace app\models;

class Users extends \lithium\data\Model {

	public function fullName($entity) {
		return $entity->firstName . ' ' . $entity->middleInitial . '. ' . $entity->lastName;
	}
}
```

If you want to add additional parameters, do so after you've specified the entity as the first:

```php
namespace app\models;

class Users extends \lithium\data\Model {
	
	public function fullName($entity, $suffix = false) {
		$name = $entity->firstName . ' ' . $entity->middleInitial . '. ' . $entity->lastName;
		
		if ($suffix) {
			$name .= ' ' . $entity->suffix;
		}
		return $name;
	}
}
```

Once this is done, use it wherever you've got access to a `Users` model instance:

```php
$firstUser = Users::first();

$firstUser->fullName();  // "Bill S. Preston"
$firstUser->fullName(true);  // "Bill S. Preston Esq."
```

