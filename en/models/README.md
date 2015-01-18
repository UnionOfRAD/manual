# Models Overview
li3 provides you with methods that your model can use when it inherits from `lithium\data\Model`. Some of these are static, some are not. As a rule of thumb, all static methods work with a collection of objects on the database, while non-static methods are tied to one single document/record. Consider the following example:

```
$post = Posts::first(array('conditions' => array('author' => 'foobar')));

$post->title = 'Hello World';
$post->save();
```

The `first()` method is static because it iterates over a bunch of entities in your database and returns the first entry where the `author` equals `foobar`. The result you get is an instance of a `Post` so all subsequent methods on it are _non-static_. The second line in the example sets the `title` and the third line saves it back to the database. This concept feels natural and also has the benefit of instantly knowing on what kind of dataset you're operating.

The response from a model method is not just a plain array but actually a record/document object (or a collection of them). This means that you can perform a variety of actions on them if you need to. Here are a few examples:

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

// Convert to a plain array
$plain = $posts->to('array');
```

Although extending models is beyond the scope of this page, here's another example so you get a glimpse of what's possible with these objects.

```
// In app/models/People.php
namespace app\models;

class People extends \lithium\data\Model {
	
	public function fullName($entity) {
		return "{$entity->firstname} {$entity->lastname}";
	}
}

// In app/views/people/index.html.php
foreach ($people as $person) {
	echo $person->fullName();
}
```

You can add functionality on the fly and provide an extra layer of abstraction around raw database records/documents with very little effort. 
This technique is in depth described in [Adding Functionality](./adding-functions-to-models.md).






