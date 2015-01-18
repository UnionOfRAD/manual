# Setting Up Models

## Introduction
Models play a key role in nearly every web application. Their main purpose is to abstract business logic and database operations from higher levels (controllers and views). They also act as a gatekeeper and—if properly implemented—make sure that only valid and allowed data gets passed through them. Models in li3 have three main purposes:

 1. Provide an abstraction layer to the underlying data source(s)
 2. Perform common data operations (like fetching and storing data)
 3. Help with validating data.

Also, li3 makes it easy to extend models so that they fit your application's needs. Thanks to the nifty autoloading mechanism, models are lazy-loaded and are only initialized when you need them. In the next sections you will learn how to use models and perform common operations on them. Later sections will provide you with a more detailed look on models like relationships between them and how to extend models to fit your application's needs.

The `Model` class is the starting point for the domain logic of your application.  Models are tasked with providing meaning to otherwise raw and unprocessed data (e.g. user profile).  Models expose a consistent and unified API to interact with an underlying datasource (e.g. MongoDB, CouchDB, MySQL) for operations such as querying, saving, updating and deleting data from the persistent storage.

Models allow you to interact with your data in two fundamentally different ways: querying,and data mutation (saving/updating/deleting). All query-related operations may be done through the static `find()` method, along with some additional utility methods provided for convenience. Classes extending the `Model` class should, conventionally, be named as Plural, CamelCase and be placed in the `models` directory. i.e. a posts model would be `model/Posts.php`.

<div class="note note-hint">
	How to <strong>save and query data</strong> is described in <a href="working-with-entites.md">Working with Entities</a>. How to <strong>validate data</strong> is described in <a href="validation.md">Validation</a> which also has information about adding custom rules and more.
</div>

## Creating a Model

li3 provides you with a general-purpose class that all your models should extend. You can find the `Model` class in the `lithium\data` namespace. If you do nothing more than extend it, you instantly get a bunch of functionality that covers basic CRUD as well as more complex tasks.

Let's say you want to store and manage blog posts in your database. According to li3 conventions, you create a new file called `Posts.php` in `app/models`. The basic structure looks like this:

```
namespace app\models;

class Posts extends \lithium\data\Model {}
```

<div class="note note-hint">
	li3 also allows model creation via the console:  You can enter <code>li3 model create Posts</code> into the command line (assuming you have configured the command line for use) and the code above will automatically be created in a file called <code>\app\models\Posts.php</code>.
</div>

## Meta-Information

Each model needs some meta-information to initialize. By default you don't have to define this
information yourself. The model class is _smart_ enough to figure all of the configuration options
out on its own - as long as you follow the frameworks conventions.

However changing the meta-information may be desired in such cases where you are createing a model which needs to access a table which doesn't follow our conventions.

The following lists all meta-information options that can be changed:

* **name**: Name of the model, defaults to the class name.
* **title**: The field that is most closely associated with being the title of each record.  Defaults to `title` or `name` if either of those fields are available.
* **key**: The field that acts as the primary key or identifier for a record.  Defaults to `id` if that field is available.  The key can also be set using the `key()` method.
* **source**: The table name or document collection being accessed.  The default is an all lowercase version of the class name with underscores as spaces. Class names like `BlogPost` and `BlogPosts` would default to the `blog_posts` table.
* **connection**: the identifier of the data connection to be used, as defined in the `\app\bootstrap\database.php` file. Defaults to `default`.

You may either define the `$_meta` property on the model class or call the model's `meta()` method to
change meta-information. All options will be merged with the defaults.

```php
class Posts extends \lithium\data\Model {

	protected $_meta = array(
		'connection' => 'legacy',
		'source' => 'tblPost',
		'key' => 'post_id'
	);
}

// ... or ...

Posts::meta('key', 'post_id');
```

## Connections

Models will look for the `'default'` connection first. If you've got more than one connection you're using, or wish a model to use an alternate, specify the connection name in the model's `$_meta` property.

```php
class Posts extends \lithium\data\Model {

	protected $_meta = array(
		'connection' => 'legacy'
	);
}
```

For connection-less models you may disable the connection altogether by setting the connection to use to `false`.

```php
class Movies extends \lithium\data\Model {

	protected $_meta = array(
		'connection' => false
	);
}
```

If you ever need to access the connection for model directly, you may do so by using the `connection()` method. The following example shows how to make use of this feature to control PDO transactions.

```php
$source = Posts::connection(); // Gives you the connected data source i.e. a `Database` object.
$pdo = $source->connection; // Gives you the underlying connection of that object.

$pdo->beginTransaction();
$pdo->commit();
$pdo->rollback();
```

## Default Query Options

In cases where you always want finders results constrained to i.e. certain conditions, default query options can be used. Default options may be defined by using the `query()` method or alternatively by defining the `$_query` property on the model class.

Specific query options overwrite default ones. As both are merged by simply using the `+` operator for arrays. Note that this can also be common pitfall.

```php
Posts::query(array(
	'conditions' => array('is_published' => true),
	'limit' => 4
));

// Will retrieve maximum of 4 results which are published.
Posts::find('all');

// Potential pitfall: will retrieve results published or not 
// for author michael. Limited to 4 results maximum.
Posts::find('all', array(
	'conditions' => array('author' => 'michael')			
));

// Will retrieve only published results for author michael.
// Limited to 4 results.
Posts::find('all', array(
	'conditions' => array('author' => 'michael', 'is_published' => true)
));
```

## Verifying Model Fields

The `hasField()` method checks to see if a particular field exists in a model's schema. This method can check a single field, or return the first field found in an array of multiple options.  The parameter is `$field`, a single field (string) or list of fields (array) to check the existence of.

## Resetting Model Instances

li3 provides the `reset()` method to reset/destroy instances of your model if that is required in your app.  This will unset the instances of the model.
