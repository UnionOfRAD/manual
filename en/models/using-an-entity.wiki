# Using An Entity
##Entity Defined
Lithium is as capable of working with document oriented data sources as it is with relational databases.  Additionally, Lithium works with other types of user-defined data sources.  Because of this, we use the term "Entity" to refer to what might be considered a document in one data source type or a record/row in another type.

## Setting Default Query Options
Lithium allows you to get/set default options and conditions for queries that are executed for a model if no query-specific options are set using the `query()` method.  This is done by passing an associative array to the $options parameter.  If you do not set an `$options` parameter, the method will return the currently set query options in an array.
The settable elements are:

* **conditions**: associative array of where conditions. Example: `'conditions' => array('status' => 'draft')`
* **fields**: Fields to be retrieved. Example: `'fields' => array('title','slug','id')`
* **order**: specify how the records are to be ordered. Example: `'order' => array('date_created' => 'DESC')`
* **limit**: Number of records to return. Example:  `'limit' => 10`
* **page**: For pagination of data (limit * offset)
* **with**: array of relationship names to include in the query

> **Shortcut**

> You can optionally set the protected `$_query` property directly in the definition of your model's class.

## Creating Entities ##
The `create()` method instantiates a new record or document object, initialized with any data passed in.

**Example:**

{{{
$post = Posts::create(array('title' => 'New post'));
echo $post->title; // echoes 'New post'
$success = $post->save();
}}}

> **Caution**

> While this method creates a new object, there is no effect on the database until the `save()` method is called.

This method can also be used to simulate loading a pre-existing object from the database, without actually querying the database:

{{{
$post = Posts::create(array('id' => $id, 'moreData' => 'foo'), array('exists' => true));
$post->title = 'New title';
$success = $post->save();
}}}

This will create an update query against the object with an ID matching `$id`. Also note that only the `title` field will be updated.

## Basic Entity Retrieval ##
 Reading data from your database (or any data source available) is one of the most basic tasks that you will perform. To make common tasks easy, Lithium provides a central method for reading data. The main (static) method used to read data is called `find()` and takes at least one parameter.

 The `find` method allows you to retrieve data from the connected data source. Within the method there are some built-in options that you can use in the `$type` parameter to specify which records you want. Custom values for this parameter can be created by using the $_finder property.

The first parameter is $type and allows you to set the finder which will be used to set the scope of data to be returned.  Built-in finders are listed below, and you can also [create custom finders](#finders).

* **all**: Retrieves all records
* **count**: Retrieve a count of all records
* **first**: Retrieve the first record
* **list**: Produces an array where the `id` field is the key and the `title` field is the value.  (See the $_meta property above for more info on `id` and `title`)

The second parameter allows you to specify options for the query.  The options available are similar to the ones that are available in the `ModelName::_query` property:

* **conditions**: associative array of where conditions.  Example: `'conditions' => array('status' => 'draft')`
* **fields**: Fields to be retrieved   Example: `'fields' => array('title','slug','id')`
* **order**: specify how the records are to be ordered  Example: `'order' => array('date_created' => 'DESC')`
* **limit**: Number of records to return  Example:  `'limit' => 10`
* **page**: For pagination of data (limit * offset)

**EXAMPLES:**

{{{
// Read all posts
$posts = Posts::find('all');

// Read the first post
$post = Posts::find('first');

// Read all posts with the newest ones first
$posts = Posts::find('all', array(
	'order' => array('created' => 'DESC')
));

// Read the only the title of the newest post
$post = Posts::find('first', array(
	'fields' => array('title'),
	'order' => array('created' => 'DESC')
));

// Read only posts where the author name is "michael"
$posts = Posts::find('all', array(
	'conditions' => array('author' => 'michael')
));
}}}

> **Tip**

> You can quickly get your data into a multidimensional array in the form of $array[keyValue][fieldName] by chaining the ->to('array') method to the end of the find() method call.

> Example: `$posts = Posts::find('all')->to('array');`

### Basic Finder Methods
Lithium also provides some additional basic methods around the `find()` method which make your code less verbose and easier to read:

{{{
// Read all posts
$posts = Posts::all();

// Read the first post
$post = Posts::first();

// Read all posts with the newest ones first
$posts = Posts::all(array('order' => array('created' => 'DESC')));
}}}

### Dynamic Finder Methods
The basic finder methods are nice, but Lithium also provides you with a set of highly dynamic methods that match against your dataset. The example below shows two different approaches to finding all the posts related to the username "Michael". The first bare approach shows how to use `find()` directly. The second example uses camelCase convention to tell Lithium to filter by a specific field name and value.

{{{
// Bare approach
$posts = Posts::find('all', array(
	'conditions' => array('username' => 'michael')
));
// Dynamic approach
$posts = Posts::findAllByUsername('michael');
}}}

Lithium also allows you to build custom finder methods to extend functionality.  This is explained in more detail in the [Adding Functions to Models](adding-functions-to-models.wiki) page.

## Updating Entities
The `update()` method allows you to update multiple records or documents with the given data, restricted by the given set of criteria (optional).

The `$data` parameter is typically an array of key/value pairs that specify the new data with which the records will be updated. For SQL databases, this can optionally be an SQL fragment representing the `SET` clause of an `UPDATE` query.  The `$conditions` parameter is an array of key/value pairs representing the scope of the records to be updated.  The `$options` parameter specifies ny database-specific options to use when performing the operation. The options parameter varies amongst the various types of data sources.  More detail is available in the source code documentation for the `delete()` methods of each data source type (Example: `\lithium\data\source\Database.php`).

**Examples**

{{{
// Change the author for all documents.
$success = Posts::update(array('author' => 'Michael'));

// Set a default title for all empty titles
Posts::update(array('title' => 'Fixme'), array('title' => ''));
}}}

## Deleting Entities
Deleting entities from your datasource is accomplished using either the `remove()` or `delete()` methods.

Use `remove()` to remove data (documents, records, etc.) based on specified conditions & options.

The `$conditions` parameter is first and is an array of key/value pairs representing the scope of the records or documents to be deleted. Example: `'conditions' => array('status' => 'draft')`.  The `$options` parameter is an array that specifies any database-specific options to use when performing the operation.  The options parameter varies amongst the various types of data sources.  More detail is available in the source code documentation for the `delete()` methods of each data source type (Example: `\lithium\data\source\Database.php`).

**Usage Examples**

{{{
// Delete all posts.
$success = Posts::remove();

// Delete all posts with an empty title.
Posts::remove(array('title' => ''));
}}}

> **WARNING!**

> Using the `remove()` method with no `$conditions` parameter specified will delete all entities in your data source.


To delete the data associated with the a specified entity, use the `delete()` method.

The parameter `$entity` is the entity to be deleted and as with `remove()`,  the `$options` parameter is an array specifies any database-specific options to use when performing the operation.  The options parameter varies amongst the various types of data sources.  More detail is available in the source code documentation for the `delete()` methods of each data source type (Example: `\lithium\data\source\Database.php`).

**Usage Example**

{{{
// Read the first post
$post = Posts::first();

// Delete the data associated with the first post
$result = Posts::delete($post);
}}}

## Saving Entities
Persisting data means that new data is being stored or updated. Before you can save your data, you have to initialize a `Model`. This can either be done with the `find()` methods shown earlier or—if you want to create a new entity with the static `create()` method.  `save()` is a method (called on record and document objects) to create or update the record or document in the database that corresponds to `$entity`.

**EXAMPLE: Saving a Record or Document**
{{{
// Create a new post, add title and author, then save.
$post         = Posts::create();
$post->title  = "My first blog post.";
$post->author = "Michael";
$post->save();

// Same as above.
$data = array(
	'title'  => "My first blog post.",
	'author' => "Michael"
);
$post = Posts::create($data)->save();

// Find the first blog post and change the author
$post = Posts::first();
$post->author = "Michael";
$post->save();
}}}

Note that `save()` also validates your data if you have any validation rules defined in your model. It returns either `true` or `false`, depending on the success of the validation and saving process. You'll normally use this in your controller like so:

{{{
public function add() {
	$post = Posts::create();
	if($this->request->data && $post->save($this->request->data)) {
		$this->redirect('Posts::index');
	}
	return compact('post');
}
}}}
This redirects the user only to the `index` action if the saving process was successful. If not, the form is rendered again and errors are shown in the view.

To override the validation checks and save anyway, you can pass the `'validate'` option:

{{{
$post->title = "We Don't Need No Stinkin' Validation";
$post->body = "I know what I'm doing.";
$post->save(null, array('validate' => false));
}}}

> **Note**

> see the `validates()` method in the Data Validation section for more information on validation.

The `$entity` parameter is the record or document object to be saved in the database. This parameter is implicit and should not be passed under normal circumstances. In the above example, the call to `save()` on the `$post` object is transparently proxied through to the `Posts` model class, and `$post` is passed in as the `$entity` parameter.  The `$data` parameter is for any data that should be assigned to the record before it is saved.

There is also an `$options` parameter that has the following settable elements.

* `'callbacks'` _boolean_: If `false`, all callbacks will be disabled before executing. Defaults to `true`.
* `'validate'` _mixed_: If `false`, validation will be skipped, and the record will be immediately saved. Defaults to `true`. May also be specified as an array, in which case it will replace the default validation rules specified in the `$validates` property of the model.
* `'events'` _mixed_: A string or array defining one or more validation _events_. Events are different contexts in which data events can occur, and correspond to the optional `'on'` key in validation rules. They will be passed to the validates() method if `'validate'` is not `false`.
* `'whitelist'` _array_: An array of fields that are allowed to be saved to this record.

## Verifying Model Fields
The `isField()` method checks to see if a particular field exists in a model's schema. This method can check a single field, or return the first field found in an array of multiple options.  The parameter is `$field`, a single field (string) or list of fields (array) to check the existence of.

## Resetting Model Instances
Lithium provides the `reset()` method to reset/destroy instances of your model if that is required in your app.  This will unset the instances of the model.