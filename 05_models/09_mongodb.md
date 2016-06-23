# MongoDB

## Introduction
Since MongoDB queries are in JSON format, they are mostly straight forward when it comes to converting
them to PHP arrays. Many examples can be found for this in the PHP manual pages for the MongoDB PECL
extension as well as other sites. However, Lithium's `Model` class abstracts things a little bit so to
clarify, we will go through a few examples here. It should also be noted that you can still access
all of the Mongo classes provided by the PECL extension through Lithium so you are not being limited.

## Queries
Let's start off with a basic query example from the main section.

```php
// Read only posts where the author name is "tom"
$posts = Posts::find('all', [
	'conditions' => ['author' => 'tom']
]);
```

**$and**    
Now, let's say you wanted to find posts from "tom" and those posts with a category of "technology"
for example. This is another basic query that can be written one of two ways. The first way is probably
obvious by now and simply requires you to add `'category' => 'technology'` to your `conditions` array.
The second way is a little lengthier but is a great way to get familiar with how you'll be making some
other, more advanced, queries.

```php
$posts = Posts::find('all', [
	'conditions' => [
		'$and' => [
			'author' => 'tom',
			'category' => 'technology'
		]
	]
]);
```

You can see how this compares with [MongoDB's documentation](http://docs.mongodb.org/manual/reference/operator/and/)
and keep in mind the same notes. MongoDB provides an implicit **and** operation when specifying a comma
separated list of expressions. Then if you need an **and** for the same field, but with two different
expressions, you'll want to use the **$and** operator again.

**$or**    
Perhaps similar to **$and** is the **$or** operator. Let's say you wanted to find all posts where the
category was "technology" or "politics" for example.

```php
$posts = Posts::find('all', [
	'conditions' => [
		'$or' => [
			'category' => 'technology',
			'category' => 'politics'
		]
	]
]);
```

Of course you can specify different fields in the **$or** array value. For example, you may have tags
on your blog posts and wish to find all posts of a specific category or tag. However, for the same
field, we have another way to write the query above.

**$in**    
MongoDB also allows us to use an **$in** or **$nin** operator to look for values that may or may not
be in an array of values that you provide. So the above query could be written as:

```php
$posts = Posts::find('all', [
	'conditions' => [
		'category' => [
			'$in' => ['technology', 'politics']
		]
	]
]);
```

Notice that it's a little different in this case. The operator comes after the field where the **$or**
operator came before. If you're ever at a loss, you can reference 10gen's documentation for MongoDB. 
They have many clear examples for forming queries. You'll get more familiar with automatically converting
the JSON to PHP array format in your head over time.

It is also good to note that there is an **$all** operator as well. This also works with an array value
and it ensures that a field value contains all elements in that array.

### Greater than, less than, and the other usual suspects...   
Of course you have a whole bunch of operators such as **$gt**, **$lt**, **$gte**, **$lte**, and so on. 
These all work similar to the query above. If you're looking to make a query to find all documents with
a created date more recent than a day from now, you could make the following query:

```php
$date = new MongoDate(strtotime('-1 day'));
$posts = Posts::find('all', [
	'conditions' => [
		'created' => [
			'$gt' => $date
		]
	]
]);
```

Note something new here. We just used the `MongoDate` class (make sure you provide the use statement
above your class declaration of course). This is because MongoDB uses UTC dates and has its own date
object. It's very easy to convert a timestamp to the proper format by using this class. It accepts
timestamps created with `strtotime()` of course so it's typically very convenient for your queries
written in PHP.

### Getting more advanced...    
MongoDB has an exhaustive set of operators and they seem to add more all the time. Essentially, you
can access complex data in a variety of ways and this is important for a schemaless database. While keeping
your data as well organized as possible is always advised for performance (and sanity) reasons, you do have
some more advanced options in case you need them.

**$type**    
For example, since we're talking about a schemaless database, we have the **$type** operator. MongoDB
does know the value type stored within each field of a document and in rare circumstances you may find
a need to query for it. You can read more in the [MongoDB docs](http://docs.mongodb.org/manual/reference/operator/type/).
So if you wanted to find all posts with a title field that contains a string (Why? I don't know, but you could).

```php
// Find all posts with a title that is a string. Type 2 is string.
$posts = Posts::find('all', [
	'conditions' => [
		'title' => [
			'$type' => 2
		]
	]
]);
```

**$exists**    
While we're on the subject of the nature of schemaless databases, and the importance to understand your
data model, let's take a look at another operator. You can make a query that only returns documents if
a field actually exists and this is a perhaps a more common to use than the **$type** operator. For example:

```php
// Find all posts where the title field exists.
$posts = Posts::find('all', [
	'conditions' => [
		'title' => [
			'$exists' => true
		]
	]
]);
```

You may use a query like this, to instead, find documents where a field does not exist in order to clean
things up. For example, you may run some queries that remove documents that are missing data or you may
make an update based on the condition that a field does not (yet) exist. Needless to say, using the
**$exists** operator will make you feel clever.

**Regular Expressions**    
Yes, MongoDB let's you query with regular expressions. However, it should be noted that this isn't
exactly the most efficient query to be making. There is a `MongoRegex` class that the PHP driver
provides, but Lithium will use this for you when you use the `like` key in your query. This is to ensure
query convention between databases (of course MySQL uses LIKE which is where we get that from).
However, we should note that the follwing query would not work if you switched the datasource to a MySQL
database because the value for the LIKE condition wouldn't know what to do with the regex.

```php
// Find all posts where the title starts with "the"
$posts = Posts::find('all', [
	'conditions' => [
		'title' => [
			'like' => '/^the/i'
		]
	]
]);

// This is also the same query...
$posts = Posts::find('all', [
	'conditions' => [
		'title' => new MongoRegex('/^the/i')
	]
]);
```

Again, there are a ton of operators and we won't cover them all here. However, you should now be able to
reference example queries in JSON and make the necessary conversions to work in Lithium.

### Querying Complex Objects
Ok, let's dive in a little deeper here. You'll often have objects in your documents in MongoDB and you
can query for any of those sub-fields in much the same way using the same operators as you do for
the top level field keys. Instead, you'll just use the `.` as a separator. Consider the following for example:

```php
$post = Posts::find('first', [
	'conditions' => [
		'author.name' => 'tom'
	]
]);
```

In this case we have a more complex author field. Instead of it just being a simple string value with a name,
we have more attributes. Perhaps name, e-mail, and website for example. When you get the document back, you
might access the value like so:

```php
// Display the author name.
echo $post->author->name;
```

**$elemMatch**    
If you need multiple conditions for a sub-object you might want to use **$elemMatch**. Consider the following example;
you want to return all posts where the author's name is "tom" and where the author's e-mail is "tom@site.com". **$elemMatch**
allows us to match an entire document within an array.

```php
$post = Posts::find('first', [
	'conditions' => [
		'author' => [
			'$elemMatch' => [
				'name' => 'tom',
				'email' => 'tom@site.com'
			]
		]
	]
]);
```

Of course for this particular query the following also works the same:

```php
$post = Posts::find('first', [
	'conditions' => [
		'author.name' =>  'tom',
		'author.email' => 'tom@site.com'
	]
]);
```

This next part is important and a little confusing. Let's assume we can have multiple authors for a post now.
Each author object in the document still has a name, e-mail address, etc. To make this easier to envision let's
also add a "location" to this sub-object. If you do not use **$elemMatch** and make a query like so:

```php
$post = Posts::find('first', [
	'conditions' => [
		'authors.name' =>  'tom',
		'authors.location' => 'Palo Alto'
	]
]);
```

You're going to get back a bunch of documents that you might not want. To help illustrate, imagine the following
document (in JSON format):

```php
{
	title: "Blog Post A",
	authors: [
		{
			name: "someone else",
			location: "Palo Alto"
		},
		{
			name: "tom",
			location: "New York"
		}
	]
}
```

In this case the query will match and return a document like this even though we have the wrong "tom" author. Why?
Because **two different sub-objects** on the "authors" field fulfilled the query conditions. Remember we didn't
specify an array key position in our query. If the query instead was like so:

```php
$post = Posts::find('first', [
	'conditions' => [
		'authors.0.name' =>  'tom',
		'authors.0.location' => 'Palo Alto'
	]
]);
```

Then the above document would not be returned. However, this kind of query presents another problem. We don't know
for sure that the author will be the first item in the array. Unless we do...And your goal was to return all
documents where the first author was "tom" from "Palo Alto" then you're ok. Assuming that isn't the case though,
you'll want to use **$elemMatch** to ensure you match a sub-object exactly.

The following query:
```php
$post = Posts::find('first', [
	'conditions' => [
		'authors' => [
			'$elemMatch' => [
				'name' => 'tom',
				'location' => 'Palo Alto'
			]
		]
	]
]);
```

Would only return posts that had an author on the "authors" field that matched both conditions. It would not
use a name from one author sub-object and location from another to fulfill the query conditions.

So again, be very careful with sub-objects and keep your schema in mind because despite the database being
schemaless, schema still does matter. Also pay attention to your indexing for performance!

