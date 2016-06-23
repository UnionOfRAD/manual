# Model Meta Information

Each model needs some meta-information to initialize. By default you don't have to define this
information yourself. The model class is _smart_ enough to figure all of the configuration options
out on its own - as long as you follow the frameworks conventions.

However changing the meta-information may be desired in such cases where you are creating a model which needs to access a table which doesn't follow our conventions.

## Defining Meta Information

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

	protected $_meta = [
		'connection' => 'legacy',
		'source' => 'tblPost',
		'key' => 'post_id'
	];
}

// ... or ...

Posts::meta('key', 'post_id');
```

## Retrieving Meta Information

```php
Posts::meta('source');
Posts::meta('key');
// an alternative for the key is:
Posts::key();
```

## Verifying Model Fields

The `hasField()` method checks to see if a particular field exists in a model's schema. This method can check a single field, or return the first field found in an array of multiple options.  The parameter is `$field`, a single field (string) or list of fields (array) to check the existence of.

