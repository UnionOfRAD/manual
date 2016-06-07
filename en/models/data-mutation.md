# Data Mutation

## Create

The `create()` method instantiates a new record or document object, initialized with any data passed in.

```php
$post = Posts::create(array('title' => 'New post'));
echo $post->title; // echoes 'New post'
```

<div class="note note-info">
	While this method creates a new object, there is no effect 
	on the database until the <code>save()</code> method is called.
</div>

This method can also be used to simulate loading a pre-existing object from the database, without actually querying the database:

```php
$post = Posts::create(array('id' => $id, 'moreData' => 'foo'), array('exists' => true));
$post->title = 'New title';
$success = $post->save();
```

This will create an update query against the object with an ID matching `$id`. Also note that only the `title` field will be updated.

## Update

The `update()` method allows you to update multiple records or documents with the given data, restricted by the given set of criteria (optional).

The `$data` parameter is typically an array of key/value pairs that specify the new data with which the records will be updated. For SQL databases, this can optionally be an SQL fragment representing the `SET` clause of an `UPDATE` query.  The `$conditions` parameter is an array of key/value pairs representing the scope of the records to be updated.  The `$options` parameter specifies any database-specific options to use when performing the operation. The options parameter varies amongst the various types of data sources.  More detail is available in the source code documentation for the `update()` methods of each data source type (Example: `\lithium\data\source\Database.php`).

```
// Change the author for all documents.
Posts::update(array('author' => 'Michael'));

// Set a default title for all empty titles
Posts::update(array('title' => 'Untitled'), array('title' => ''));
```

## Delete

Deleting entities from your data source is accomplished using either the `delete()` 
entity method or the static `remove()` method.

Ususally you should first retrieve the entity to-be-deleted, probably do some safety check
and then delete it.

```php
$post = Posts::first(); // Read the first post.
$post->delete(); // Delete first post.
```

To delete multiple records you first retrieve a set of entities
then invoke the delete method on all of them. As the result 
of the find call here is actually a `Collection` any method 
called on the collection will be dispatched to each contained entity.


```php
// Select all drafted posts, $posts is a Collection.
$posts = Posts::find('all', array(
	'conditions' => array('is_draft' => true)			
));

// Iterates over each post in collection and deletes it.
$posts->delete(); 

// ... is the same as: 
foreach ($posts as $post) {
	$post->delete();
}
```

As an alternative to quickly remove massive sets of entities `remove()` together with
conditions can be used. The `$conditions` parameter is first and is an array of key/value
pairs representing the scope of the records or documents to be deleted.

```
// Delete all posts!!
$success = Posts::remove();

// Delete drafted posts only.
Posts::remove(array('is_draft => true));
```

<div class="note note-caution">
	Using the <code>remove()</code> method with no <code>$conditions</code> 
	parameter specified will delete all entities in your data source.
</div>


