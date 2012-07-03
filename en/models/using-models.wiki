# Using Models

## Introduction

Models play a key role in nearly every web application. Their main purpose is to abstract business logic and database operations from higher levels (controllers and views). They also act as a gatekeeper and—if properly implemented—make sure that only valid and allowed data gets passed through them. Models in Lithium have three main purposes:

 1. Provide an abstraction layer to the underlying data source(s),
 2. Make common operations (like fetching and storing data) easy to use and
 3. Help with validating data.

Also, Lithium makes it easy to extend models so that they fit your application's needs. Thanks to the nifty autoloading mechanism, models are lazy-loaded and are only initialized when you need them. In the next sections you will learn how to use models and perform common operations on them. Later sections will provide you with a more detailed look on models like relationships between them and how to extend models to fit your application's needs.

## Model Creation and Configuration

Lithium provides you with a general-purpose class that all your models should extend. You can find the `Model` class in the `lithium\data` namespace. If you do nothing more than extend it, you instantly get a bunch of functionality that covers basic CRUD as well as more complex tasks.

Let's say you want to store and manage blog posts in your database. According to Lithium conventions, you create a new file called `Posts.php` in `app/models`. The basic structure looks like this:

{{{
<?php
namespace app\models;
class Posts extends \lithium\data\Model {
}
?>
}}}

If you don't specify otherwise, your model will use the `default` connection specified in `app/config/connections.php` (along with some other default configuration settings). All these defaults are stored in the model's `$_meta` variable and can be overridden if the defaults don't fit your needs. Let's say you want to use the `backup` connection instead of the default one.

{{{
<?php
namespace app\models;
class Posts extends \lithium\data\Model {
	protected $_meta = array(
		'connection' => 'backup'
	);
}
?>
}}}

Once your model's `$_meta` property has been configured, Lithium merges it with the default settings at runtime. Because the `Post` model has a specified connection, the `backup` connection is used instead of the default.

Now that we've got you up and running, let's talk about some basic concepts before we dig into reading and writing data.

## Concepts

Lithium provides you with methods that your model can use when it inherits from `lithium\data\Model`. Some of these are static, some are not. As a rule of thumb, all static methods work with a collection of objects on the database, while non-static methods are tied to one single document/record. Consider the following example:

{{{
<?php
$post = Posts::first(array('conditions' => array('author' => 'foobar')));

$post->title = "Hello Lithium!";
$post->save();
?>
}}}

The `first()` method is static because it iterates over a bunch of entities in your database and returns the first entry where the `author` equals `foobar`. The result you get is an instance of a `Post` so all subsequent methods on it are _non-static_. The second line in the example sets the `title` and the third line saves it back to the database. This concept feels natural and also has the benefit of instantly knowing on what kind of dataset you're operating.

The response from a model method is not just a plain array but actually a record/document object (or a collection of them). This means that you can perform a variety of actions on them if you need to. Here are a few examples:

{{{
<?php
// Find all Posts
$posts = Posts::all();

// Get the first and last post of the collection
$first = $posts->first();
$last  = $posts->last();

// Iterate over all posts and print out the title
foreach($posts as $post) {
	echo $post->title;
}

// Convert to a plain array
$plain = $posts->to('array');
?>
}}}

Although extending models is described in a later chapter, here's another example so you get a glimpse of whats possible with these objects.

{{{
<?php
// In app/models/Posts.php
namespace app\models;
class Person extends \lithium\data\Model {
	public function fullName($entity) {
		return $entity->firstname." ".$entity->lastname;
	}
}

// In app/views/posts/index.html.php
foreach($people as $person) {
	echo $person->fullName();
}
?>
}}}

You can add functionality on the fly and provide an extra layer of abstraction around raw database records/documents with very little effort.

Let's get our hands dirty with some code.

## Basic CRUD

CRUD is an abbreviation and stands for Create, Read, Update and Delete. CRUD describes the basic operations that most applications perform against the database. As with many database abstraction layers, Lithium provides you with a rich set of methods that help you with such database operations. At the end of this chapter, you should be able to create and modify datasets regardless of the underlying storage engine.

### Retrieving Data

Reading data from your database (or any datasource available) is one of the most basic tasks that you will perform. To make common tasks easy, Lithium provides a central method for reading data. The main (static) method used to read data is called `find()` and takes at least one parameter. 

The first argument tells the method how many records you want to fetch. The optional second argument is an array of params that let you narrow down your search results. Let's go through a bunch of examples:

{{{
<?php
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
?>
}}}

Lithium also provides wrappers around the `find()` method which make your code less verbose and easier to read:

{{{
<?php
// Read all posts
$posts = Posts::all();

// Read the first post
$post = Posts::first();

// Read all posts with the newest ones first
$posts = Posts::all(array('order' => array('created' => 'DESC')));
?>
}}}

These basic wrappers are nice, but Lithium also provides you with a set of highly dynamic wrappers that match against your dataset. The example below shows two different approaches to finding all the posts related to the username "Michael". The first bare approach shows how to use `find()` directly. The second example uses camelCase convention to tell Lithium to filter by a specific field name and value.

{{{
<?php
// Bare approach
$posts = Posts::find('all', array(
	'conditions' => array('username' => 'michael')
));
// Dynamic approach
$posts = Posts::findAllByUsername('michael');
?>
}}}

### Saving Data

Persisting data means that new data is being stored or updated. Before you can save your data, you have to initialize a `Model`. This can either be done with the `find()` methods shown earlier or—if you want to create a new record—with the static `create()` method. You 
then have various ways to add or modify your data. Here are some examples:

{{{
<?php
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
?>
}}}

Note that `save()` also validates your data if you have any validation rules defined in your model. It returns either `true` or `false`, depending on the success of the validation and saving process. You'll normally use this in your controller like so:

{{{
<?php
public function add() {
	$post = Posts::create();
	if($this->request->data && $post->save($this->request->data)) {
		$this->redirect('Posts::index');
	}
	return compact('post');
}
?>
}}}

This redirects the user only to the `index` action if the saving process was successful. If not, the form is rendered again and errors are shown in the view.

### Updating Data

The last section showed you how to save documents/records in your database. If these records existed previously, you are also _updating_ them. If you have to update a large batch of data at once, fetching all records is pretty inefficient. Therefore, Lithium also features the static `update()` method which updates records directly in your database (similar to the sql `UPDATE` command). 

The first argument provides the data to change, the second (optional) one is a set of conditions to narrow down your selection. You can pass the same conditions as you would with `find()`. Here are some examples:

{{{
<?php
// Change the author for all documents.
$success = Posts::update(array('author' => 'Michael'));

// Set a default title for all empty titles
Posts::update(array('title' => 'Fixme'), array('title' => ''));
?>
}}}

### Deleting Data

Removing data works similar to updating data, the big difference is that you provide a set of conditions to delete a subset of your database entries.

{{{
<?php
// Delete all posts.
$success = Posts::remove();

// Delete all posts with an empty title.
Posts::remove(array('title' => ''));
?>
}}}

Be careful with this. If you don't provide any arguments to `remove()`, then all documents/rows in your 
database will be deleted!