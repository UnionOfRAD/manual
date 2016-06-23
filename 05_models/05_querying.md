# Querying 

Reading data from your database (or any datasource available) is a common
tasks that you will perform. All query-related operations may be done through
the static `find()` method, along with some additional utility methods provided
for convenience.

```php
// Read all posts.
$posts = Posts::find('all');

// Read the first post.
$post = Posts::find('first');

// Read all posts with the newest ones first.
$posts = Posts::find('all', [
	'order' => ['created' => 'DESC']
]);

// Read only the title of the newest post.
$post = Posts::find('first', [
	'fields' => ['title'],
	'order' => ['created' => 'DESC']
]);

// Read only posts where the author name is "michael".
$posts = Posts::find('all', [
	'conditions' => ['author' => 'michael']
]);
```

<div class="note note-caution">
	The framework protects against injection attacks by quoting
	condition values. Other options i.e. <code>'fields'</code> are
	however <strong>not</strong> automatically quoted. Read more about the topic 
	and potential countermeasures in the <a href="../security">Security</a> chapter.
</div>

## Built-in Finders
 
`find()` utilizes _finders_ to do most of its work. The first parameter to `find()`
selects the type of finder you want to use. The second parameter provides options to the
finder. 

There are four built-in finders. The most commonly used ones are `'all'` and `'first'`.

<div class="note note-hint">
	You can also create your own custom finders via `finder()`.
</div>

- `'all'`: Returns a collection of records matching the conditions.
- `'first'`: Returns the first record matching the conditions.
- `'count'`: Returns an integer count of all records matching the conditions.
- `'list'`: Returns a one dimensional array, where the key is the primary
  key and the value the title of the record (a `'title'` field is required 
  for this). A result may look like: `array(1 => 'Foo', 2 => 'Bar')`.

All bultin-in finders take the same set of options to control which and
how many records they return.

## Using Conditions

The `'conditions'` option allows you to provide conditions for the query
i.e. `'array('is_published' => true)`.  The following example finds all
posts author name is `michael`.

```php
$posts = Posts::find('all', [
	'conditions' => ['author' => 'michael']
]);
```

By default the conditions are AND'ed together so the following
example would require each post to be published **and** authored 
by `michael`.

```php
$posts = Posts::find('all', [
	'conditions' => [
		'author' => 'michael',
		'is_published' => true
	]
]);
```

To OR conditions together use the following syntax.

```php
$posts = Posts::find('all', [
	'conditions' => [
		'or' => [
			'author' => 'michael',
			'is_published' => true
		]
	]
]);
```

To find all records from a set of authors use the array
syntax.

```php
$posts = Posts::find('all', [
	'conditions' => [
		'author' => ['michael', 'nate']
	]
]);
```

## Ordering

The `'order'` option allows to control the order in which the data will be returned, i.e.
`'created ASC'` sorts by created date in ascending order, `'created DESC'` sorts
the same field in descending order. To sort by multiple fields use the array
syntax `array('title' => 'ASC', 'id' => 'ASC)`.

The following are equal groups:
```php
Posts::find('all', ['order' => 'title']);
Posts::find('all', ['order' => 'title ASC']);
Posts::find('all', ['order' => ['title']]);
Posts::find('all', ['order' => ['title' => 'ASC']]);

Posts::find('all', ['order' => ['title', 'id']]);
Posts::find('all', ['order' => ['title' => 'ASC', 'id' => 'ASC']]);
```

## Restricting Returned Fields

The `'fields'` option allows to specify the fields that should be retrieved. By default
all fields are retrieved. To optimize query performance, limit to just the ones actually
needed.

```php
Posts::find('all', [
	'fields' => ['title', 'author']			
]);
```

## Limiting Number of Records

The `'limit'` option allows to limit the set of returned records to a maximum number.

```php
Posts::find('all', [
	'limit' => 10			
]);
```

## Pagination

The `'page'` option allows to paginate data sets. It specifies the page of the set
together with the limit option specifying the number of records per page. The first
page starts at `1`.

```php
$page1 = Posts::find('all', [
	'page' => 1,
	'limit' => 10	
]);
$page2 = Posts::find('all', [
	'page' => 2,
	'limit' => 10	
]);
```

## Shorthands

li3 also provides some additional basic methods around the `find()` method which
make your code less verbose and easier to read:

```php
// Shorthand for find first by primary key.
Posts::find(23);

Posts::all();
```

<div class="note">
	Note `list` is a keyword in PHP and so cannot be called magically like other finders.
</div>

The example below shows two different approaches to finding all the posts related to the
username "Michael". The first bare approach shows how to use `find()` directly. The second
example uses camelCase convention to tell li3 to filter by a specific field name and
value.

```php
$posts = Posts::findAllByUsername('michael');
// ... is functionally equal to ...
$posts = Posts::find('all', [
	'conditions' => ['username' => 'michael']
]);
```

## Custom Finders

The basic finders are nice, but the framework also provides you with a set of highly
dynamic methods that match against your dataset. It allows you to build
custom _finders_ to extend functionality.

As you use your models, you might start to wish for a shortcut. For example,
instead of having to do this repeatedly:

```php
$recentComments = Comments::find('all', [
	'conditions' => [
		'created' => [
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		]
	]
]);
```

You could create a custom finder method that packages the specified conditions into a one-liner:

```php
$recentComments = Comments::find('recent');

// or, as a "magic" method:

$recentComments = Comments::recent();
```

At a basic level, this is done by utilizing the `finder()` method of the model.
You call `finder()` and supply the name of the finder, along with a definition
so li3 knows how to form the query. The definition in this simple case looks
just like the query array we supplied to `find()` earlier:

```php
Comments::finder('recent', [
	'conditions' => [
		'created' => [
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		]
	]
]);
```

Some finder implementations might require a little processing in addition to a default set
of conditions. In that case, you can define a finder using a closure that will be called
as part of find chaining. In this use case, you supply the name of the finder along with a
closure that looks much like a filter definition:

<div class="note note-version">
	The syntax for filter and thus for finders has changed in 1.1. 
	Below you find the new syntax. The old one receives 3 parameters
	(`$self`, `$params`, `$chain`).
</div>

```php
Comments::finder('recentCategories', function($params, $next){
	// Set up default conditions
	$defaults = [
		'created' => [
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		]
	];
	
	// Merge with supplied params
	$params['options']['conditions'] = $defaults + (array) $params['options']['conditions'];
	
	// Do a bit of reformatting
	$results = [];
	foreach ($next($params) as $entity) {
		$results[] = $entity->categoryName;
	}
	
	// Returns an array of recent categories given the supplied query params.
	return $results;
});
```

## Rich Returned Values

The response from a model method is not just a plain array but actually an
entity object (or a collection of them). This means that you can perform a
variety of actions on them if you need to. Here are a few examples:

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

## Default Query Options

In cases where you always want finders results constrained to i.e. certain conditions,
default query options can be used. Default options may be defined by using the `query()`
method or alternatively by defining the `$_query` property on the model class.

Specific query options overwrite default ones. As both are merged by simply using the `+`
operator for arrays. Note that this can also be a common pitfall.

```php
Posts::query([
	'conditions' => ['is_published' => true],
	'limit' => 4
]);

// Will retrieve maximum of 4 results which are published.
Posts::find('all');

// Potential pitfall: will retrieve results published or not 
// for author michael. Limited to 4 results maximum.
Posts::find('all', [
	'conditions' => ['author' => 'michael']			
]);

// Will retrieve only published results for author michael.
// Limited to 4 results.
Posts::find('all', [
	'conditions' => ['author' => 'michael', 'is_published' => true]
]);
```

