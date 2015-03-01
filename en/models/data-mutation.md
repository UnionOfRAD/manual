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

The `$data` parameter is typically an array of key/value pairs that specify the new data with which the records will be updated. For SQL databases, this can optionally be an SQL fragment representing the `SET` clause of an `UPDATE` query.  The `$conditions` parameter is an array of key/value pairs representing the scope of the records to be updated.  The `$options` parameter specifies ny database-specific options to use when performing the operation. The options parameter varies amongst the various types of data sources.  More detail is available in the source code documentation for the `update()` methods of each data source type (Example: `\lithium\data\source\Database.php`).

```
// Change the author for all documents.
Posts::update(array('author' => 'Michael'));

// Set a default title for all empty titles
Posts::update(array('title' => 'Untitled'), array('title' => ''));
```

## Delete

Deleting entities from your datasource is accomplished using either the `remove()` or `delete()` methods.

Use `remove()` to remove data (documents, records, etc.) based on specified conditions & options.

The `$conditions` parameter is first and is an array of key/value pairs representing the scope of the records or documents to be deleted. Example: `'conditions' => array('status' => 'draft')`.  The `$options` parameter is an array that specifies any database-specific options to use when performing the operation.  The options parameter varies amongst the various types of data sources.  More detail is available in the source code documentation for the `delete()` methods of each data source type (Example: `\lithium\data\source\Database.php`).

```
// Delete all posts.
$success = Posts::remove();

// Delete all posts with an empty title.
Posts::remove(array('title' => ''));
```

<div class="note note-caution">
	Using the <code>remove()</code> method with no <code>$conditions</code> 
	parameter specified will delete all entities in your data source.
</div>

To delete the data associated with the a specified entity, use the `delete()` method.

The parameter `$entity` is the entity to be deleted and as with `remove()`,  the `$options` parameter is an array specifies any database-specific options to use when performing the operation.  The options parameter varies amongst the various types of data sources.  More detail is available in the source code documentation for the `delete()` methods of each data source type (Example: `\lithium\data\source\Database.php`).

```php
// Read the first post
$post = Posts::first();

// Delete the data associated with the first post
$result = Posts::delete($post);
```


