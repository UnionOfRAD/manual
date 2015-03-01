# Querying 

Reading data from your database (or any data source available) is one of the most basic tasks that you will perform. To make common tasks easy, li3 provides a central method for reading data. The main (static) method used to read data is called `find()` and takes at least one parameter.

The `find` method allows you to retrieve data from the connected data source. Within the method there are some built-in options that you can use in the `$type` parameter to specify which records you want. Custom values for this parameter can be created by using the $_finder property.

The first parameter is $type and allows you to set the finder which will be used to set the scope of data to be returned.  Built-in finders are listed below, and you can also create custom finders.

* **all**: Retrieves all records.
* **count**: Retrieve a count of all records.
* **first**: Retrieve the first record.
* **list**: Produces an array where the `id` field is the key and the `title` field is the value.

The second parameter allows you to specify options for the query:

* **conditions**: Associative array of conditions.  
* **fields**: Fields to be retrieved.
* **order**: Specify how the records are to be ordered.
* **limit**: Number of records to return.
* **page**: For pagination of data (equals limit * offset).

```php
// Read all posts.
$posts = Posts::find('all');

// Read the first post.
$post = Posts::find('first');

// Read all posts with the newest ones first.
$posts = Posts::find('all', array(
	'order' => array('created' => 'DESC')
));

// Read the only the title of the newest post.
$post = Posts::find('first', array(
	'fields' => array('title'),
	'order' => array('created' => 'DESC')
));

// Read only posts where the author name is "michael".
$posts = Posts::find('all', array(
	'conditions' => array('author' => 'michael')
));
```

<div class="note note-caution">
	The framework protects against injection attacks by quoting
	condition values. Other options i.e. <code>'fields'</code> are
	however <strong>not</strong> automatically quoted. Read more about the topic 
	and potential countermeasure in the <a href="../security">Security</a> chapter.
</div>

## Rich Returned Values

The response from a model method is not just a plain array but actually an entity object (or a collection of them). This means that you can perform a variety of actions on them if you need to. Here are a few examples:

```
// Find all Posts
$posts = Posts::all();

// Get the first and last post of the collection
$first = $posts->first();
$last  = $posts->last();

// Iterate over all posts and print out the title
foreach ($posts as $post) {
	echo $post->title;
}
```

Entities can be easily converted into other formats.

```php
Posts::find('all')->data();
Posts::find('all')->to('array');
```

## Basic Finder Methods

li3 also provides some additional basic methods around the `find()` method which make your code less verbose and easier to read:

```
// Read all posts
$posts = Posts::all();

// Read the first post
$post = Posts::first();

// Read all posts with the newest ones first
$posts = Posts::all(array('order' => array('created' => 'DESC')));
```

## Dynamic Finder Methods

The basic finder methods are nice, but li3 also provides you with a set of highly dynamic methods that match against your dataset. The example below shows two different approaches to finding all the posts related to the username "Michael". The first bare approach shows how to use `find()` directly. The second example uses camelCase convention to tell li3 to filter by a specific field name and value.

```php
// Bare approach
$posts = Posts::find('all', array(
	'conditions' => array('username' => 'michael')
));
// Dynamic approach
$posts = Posts::findAllByUsername('michael');
```

## Custom Finder Methods

The framework also allows you to build custom finder methods to extend functionality.  

Models ship with a number of default "finder" methods:

```php
$users      = Users::find('all');
$oneUser    = Users::find('first');
$keyedArray = Users::find('list');
```

As you use your models, you might start to wish for a shortcut. For example, instead of having to do this repeatedly:

```php
$recentComments = Comments::find('all', array(
	'conditions' => array(
		'created_on' => array(
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		)
	)
));
```

You could create a custom finder method that packages the specified conditions into a one-liner:

```php
$recentComments = Comments::find('recent');

// or, as a "magic" method:

$recentComments = Comments::recent();
```

At a basic level, this is done by utilizing the `finder()` method of the model. You call `finder()` and supply the name of the finder, along with a definition so li3 knows how to form the query. The definition in this simple case looks just like the query array we supplied to `find()` earlier:

```php
Comments::finder('recent', array(
	'conditions' => array(
		'created_on' => array(
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		)
	)
));
```

Some finder implementations might require a little processing in addition to a default set of conditions—somewhat like the . In that case, you can define a finder using a closure that will be called as part of li3's find chaining. In this use case, you supply the name of the finder along with a closure that looks much like a filter definition:

```php
Comments::finder('recentCategories', function($self, $params, $chain){
	
	// Set up default conditions
	$defaults = array(
		'created_on' => array(
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		)
	);
	
	// Merge with supplied params
	$params['options']['conditions'] = $defaults + (array) $params['options']['conditions'];
	
	// Do a bit of reformatting
	$results = array();
	foreach ($chain->next($self, $params, $chain) as $entity) {
		$results[] = $entity->categoryName;
	}
	
	// Returns an array of recent categories given the supplied query params.
	return $results;
});
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


