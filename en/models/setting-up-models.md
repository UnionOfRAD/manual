# Setting Up Models

## Introduction
Models play a key role in nearly every web application. Their main purpose is to abstract business logic and database operations from higher levels (controllers and views). They also act as a gatekeeper and—if properly implemented—make sure that only valid and allowed data gets passed through them. Models in Lithium have three main purposes:

 1. Provide an abstraction layer to the underlying data source(s)
 2. Perform common data operations (like fetching and storing data)
 3. Help with validating data.

Also, Lithium makes it easy to extend models so that they fit your application's needs. Thanks to the nifty autoloading mechanism, models are lazy-loaded and are only initialized when you need them. In the next sections you will learn how to use models and perform common operations on them. Later sections will provide you with a more detailed look on models like relationships between them and how to extend models to fit your application's needs.

The `Model` class is the starting point for the domain logic of your application.  Models are tasked with providing meaning to otherwise raw and unprocessed data (e.g. user profile).  Models expose a consistent and unified API to interact with an underlying datasource (e.g. MongoDB, CouchDB, MySQL) for operations such as querying, saving, updating and deleting data from the persistent storage.

Models allow you to interact with your data in two fundamentally different ways: querying,and data mutation (saving/updating/deleting). All query-related operations may be done through the static `find()` method, along with some additional utility methods provided for convenience. Classes extending the `Model` class should, conventionally, be named as Plural, CamelCase and be placed in the `models` directory. i.e. a posts model would be `model/Posts.php`.

## Creating a Model
Lithium provides you with a general-purpose class that all your models should extend. You can find the `Model` class in the `lithium\data` namespace. If you do nothing more than extend it, you instantly get a bunch of functionality that covers basic CRUD as well as more complex tasks.

Let's say you want to store and manage blog posts in your database. According to Lithium conventions, you create a new file called `Posts.php` in `app/models`. The basic structure looks like this:

{{{

namespace app\models;
class Posts extends \lithium\data\Model {
}

}}}

> **Command Line Shortcut**

>Lithium also allows model creation via the console:  You can enter `li3 model create Posts` into the command line (assuming you have configured the command line for use) and the code above will automatically be created in a file called `\app\models\Posts.php`

## Setting Model Options
The `meta()` method is used to set common options.  It allows the user to specify non-default settings for connecting to a data source (table).  If you do not set any parameters, the method will return the currently set model options stored in the `$_meta` property in an array. Options are:

* **name**: Name of the model, defaults to the class name.
* **title**: The field that is most closely associated with being the title of each record.  Defaults to `title` or `name` if either of those fields are available.
* **key**: The field that acts as the primary key or identifier for a record.  Defaults to `id` if that field is available.  The key can also be set using the `key()` method.
* **source**: The table name or document collection being accessed.  The default is an all lowercase version of the class name with underscores as spaces. Class names like `BlogPost` and `BlogPosts` would default to the `blog_posts` table.
* **connection**: the identifier of the data connection to be used, as defined in the `\app\bootstrap\database.php` file.   Defaults to `default`.

You do not have to configure all elements.  The default values will be used for any element that you do not specify.  One way to set options is to call the meta method that will override and/or merge with the default settings.

> **Shortcut**

>You can optionally just set the protected `$_meta` property directly in the definition of your model's class.

**SHORTCUT EXAMPLE**:
{{{
namespace app\models;
class Posts extends \lithium\data\Model {
	protected $_meta = array(
		'connection' => 'legacy',
		'source' => 'tblPost',
		'key' => 'post_id'
	);
}
}}}

## Model Data Validation
An important part of describing the business logic of a model class is defining the validation rules. In Lithium models, rules are defined through the `$validates` class property, and are used by the `validates()` method before saving to verify the correctness of the data being sent to the backend data source.

Note that these are application-level validation rules, and do not interact with any rules or constraints defined in your data source. If such constraints fail,  an exception will be thrown by the database layer. The `validates()` method only checks against the rules defined in application code.  This method uses the `Validator` class to perform [data validation](validation.md). An array representation of the entity object to be tested is passed to the `check()` method, along with the model's validation rules. Any rules defined in the `Validator` class can be used to validate fields.

The `$entity` parameter specifies the model entity to validate, which is typically either a `Record` or `Document` object. In the  example below, the `$entity` parameter is equal to the `$post` object instance.

The options parameter has the following settable elements:

* `'rules'` _array_: If specified, this array will _replace_ the default validation rules defined in `$validates`.
* `'events'` _mixed_: A string or array defining one or more validation _events_. Events are different contexts in which data events can occur, and correspond to the optional `'on'` key in validation rules. For example, by default, `'events'` is set to either `'create'` or `'update'`, depending on whether `$entity` already exists. Then, individual rules can specify `'on' => 'create'` or `'on' => 'update'` to only be applied at certain times. Using this parameter, you can set up custom events in your rules as well, such as `'on' => 'login'`. Note that when defining validation rules, the `'on'` key can also be an array of multiple events.

**Usage Example**

{{{
$post = Posts::create($data);
$success = $post->validates();
}}}

> **Tip**

> See the section on [validation](validation.md) in this user manual for more information about adding custom rules, or override built-in rules.