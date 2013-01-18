# The Quintessential Blog Tutorial

Thanks for trying out Lithium! This guide is meant for PHP users who are looking to get a good idea of what Lithium can do, and what developing an application using the framework is like. Diving into the code and creating a simple example application is a great way to test the waters and get a good feel for rapid application development, Lithium style.

## Setting Up the Lab

First on the checklist is getting a fresh copy of Lithium. If you've not done this already, check out our [Getting Started](http://lithify.me/docs/manual/getting-started/installation.wiki) guide. Make sure to follow each of the steps in the guide carefully.

Once you've got Lithium up and running, we'll need to get some sort of persistent storage layer in place. Traditionally this would be done using some sort of SQL database like MySQL. For this example however, we're going to use something a little different: MongoDB. Using a different setup will show you how flexible the framework is in using different engines.

MongoDB is a hybrid solution, made from the good parts of both key/value data stores and traditional RDBMS systems. Setup is fast and easy.

The easiest way to get started is by downloading a binary from the MongoDB website. Check their [downloads page](http://www.mongodb.org/display/DOCS/Downloads), and fetch the binary that corresponds to your platform.

Next, perform a bit of setup by creating a data directory for Mongo. Do this by creating a `/data/db` directory on your system (`C:\data\db` for our Windows friends).

Finally, start up the database engine. Run `/path/to/mongodb/bin/mongod`, and wait for the initialization information to finish showing. If you run into any problems along the way, MongoDB's [Getting Started guide](http://www.mongodb.org/display/DOCS/Getting+Started) is a great place to get some help.

For MongoDB + PHP goodness, you'll need to install the Mongo PECL module. For most systems [(not Windows](http://www.mongodb.org/display/DOCS/Installing+the+PHP+Driver#InstallingthePHPDriver-WindowsInstall)), this is as easy as:

	sudo pecl install mongo

Once that's finished successfully, make sure to add `extension=mongo.so` to your `php.ini` file.

One last thing: better error handling and debug/production settings are slated for an upcoming release. Until then, you might want to square away a spot in `/app/config/bootstrap.php` for making your own error display decisions. For now, I'd suggest you enable the display of errors for your PHP installation or by using `ini_set()` to enable it temporarily while we're developing.

	ini_set("display_errors", 1);

## Connection Setup

Now that we have a database up and running, let's inform Lithium about the new setup. Do this by creating a new connection, in `/app/config/bootstrap/connections.php`.

Remove any connections that exist in the file, then add a new connection for our MongoDB setup:

	// MongoDB Connection
	Connections::add('default', array('type' =>  'MongoDb', 'database' => 'blog', 'host' => 'localhost'));
	
The first parameter just names the connection something that can be read by people. Naming the connection `'default'` means that our Lithium models will use this connection unless otherwise specified.

The last parameter is used for connection configuration.  First specify the`'type'` as either the name of a class (as in the above example) or the namespace containing the `'adapter'`. In this case, we tell Lithium we want to use a MongoDB database called `'blog'`.

## MVC Starts with M

Let's create your first Lithium model. This model will handle domain logic for blog posts. Let's create a new file at `/app/models/Posts.php`. Because we're relying on convention to do the heavy (and monotonous) lifting, the model file itself is short and simple.

{{{
<?php

namespace app\models;

class Posts extends \lithium\data\Model {
}

?>
}}}
	
There's a lot going on (for free!) in the background here. First, Lithium knows we're using our default connection because we haven't specified otherwise. Secondly, since our model is named `Posts`, it'll use a MongoDB collection called 'posts'.

Wait, what? What about the schema setup? Actually MongoDB doesn't require you set it up first - it just waits until you want to insert or query a collection, and handles the request appropriately.

## In Control

The controller setup is just as simple when getting started. Create a new file at `/app/controllers/PostsController.php` and fill it with the following:

{{{
<?php

namespace app\controllers;

class PostsController extends \lithium\action\Controller {
}

?>
}}}

You may start noticing a trend: filenames are CamelCase, as are classnames. File paths also match their respective namespace, and are under_scored.

Let's go ahead and create an initial action as well. Create a new `index()` function in your newly created controller. Before we get crazy with model goodness, let's just set up a simple action that pushes data to the view. Here's how it's done:

{{{
<?php

namespace app\controllers;

class PostsController extends \lithium\action\Controller {

	public function index() {
		return array('foo' => 'bar', 'title' => 'Posts');
	}
}

?>
}}}

Lithium actions send data to the view by returning an associative array that determines the respective view's variables. In the example above, our view will have access to the string `'bar'` in `$foo` and `'Posts'` in `$title` (`$title` will be used to set the page's title tag). This setup is also handy for fans of the [`compact()` function](http://php.net/compact). If I have a series of variables I'd like to pass along, it's as easy as:

	return compact('lions', 'tigers', 'bears');

## What a View

Let's see what that looks like in a view. 

Start by creating a new file at `/app/views/posts/index.html.php` (you'll need to create the posts directory). Let's start simple, and print out the data we so carefully crafted in our controller:

	Lithium is less dense than <?=$foo;?>ium.

What you're seeing here in this view is the default and preferred way to output data to an HTML page. Using the `<?= ?>` syntax automatically escapes output and keeps you safe from the legions of attacks based from unescaped output. If you really need it, there's always `<?php echo ?>`. The view code you're seeing doesn't end up as short tags when it gets to PHP's parser: Lithium's view transparently rewrites these tags to automatically escape your view output &mdash; so don't worry about your PHP installation or short tag handling.

You can now view your Posts index page by viewing `/posts/` in your browser.

## First Post!

Now that we've got our requests running smoothly, let's get the model more involved. Our first step will be to create a new action that handles the addition of new posts. To do that, we'll create a new action called `add()` and fill it with HTML form goodness. First, the new (empty) action in our `PostsController`:

	public function add() {
		
	}

Next, create a new file at `/app/views/posts/add.html.php`:
{{{
<?=$this->form->create(); ?>
	<?=$this->form->field('title');?>
	<?=$this->form->field('body', array('type' => 'textarea'));?>
	<?=$this->form->submit('Add Post'); ?>
<?=$this->form->end(); ?>
}}}
This view code sets up a simple HTML form, using a view layer assistance class called FormHelper. Don't stress the details of what the helper is doing at this point - what it outputs is most important for now.

One note: because the call to `$this->form->create()` doesn't include any parameter, Lithium assumes you mean the `add()` method of the current controller. In this case, it's pointed to `/posts/add`, just as we need.

Let's move back to the controller, and handle the data the HTML form is sending us. Here's what our `add()` action should now look like:

{{{
public function add() {
	$success = false;

	if ($this->request->data) {
		$post = Posts::create($this->request->data);
		$success = $post->save();
	}
	return compact('success');
}
}}}

There's a few things that we're doing here. Early on, we're checking to see if there's any data in the response. In this case, it will be filled with our form data. Next, we create a new Post model with the request data. To give you a little more of an idea what's going on here, the following pieces of code are equivalent:

{{{
$post = Posts::create(array('title' => 'First Post', 'body' => 'Body text!'));

// or:

$post = Posts::create();
$post->title = 'First Post';
$post->body = 'Body Text!';
}}}

At this stage the controller knows nothing about the model, `Posts::create()`, in the action; so we must tell it. At the beginning of our file, and _after_ the namespace declaration, we add a `use app\models\Posts;` line:

{{{
<?php

namespace app\controllers;

use app\models\Posts;

class PostsController extends \lithium\action\Controller {

	// ...

}

?>
}}}

Once we've handed the Post model the data, we call `save()` and eventually return the result of the save operation to the view. We can use that data in the view to show some sort of status message like so:

{{{
<?php if ($success): ?>
	<p>Post Successfully Saved</p>
<?php endif; ?>
}}}
	
## Checking the Post

Now that we've stocked our database, let's use our Post model to pull the data. Let's rewrite the `index()` action in the `PostsController` to pull a list of posts and hand it to the view:

{{{
	public function index() {
		$posts = Posts::all();
		return compact('posts');
	}
}}}

The object returned from `Post:all()` - which is equivalent to `Posts::find('all')` - is a Lithium `Document` object. The `Document` object is a response object that handles MongoDB responses more appropriately, since it's not quite a traditional RDBMS. Don't worry too much about this now: just understand that what you're getting back isn't plain array-based data.

If for some reason you need to inspect the data (for debugging purposes, for example), you can use the `to()` method on the document object. 

	var_dump($posts->data());
	
	// Output
	
	[0]=>
	  array(3) {
	    ["_id"]=>
	    string(24) "4afddab78bd3c34f41757396"
	    ["title"]=>
	    string(11) "First Post!"
	    ["body"]=>
	    string(7) "Woooooo"
	  }...

If you're curious, you might also try `$posts->to('json')` to see some additional options. To enable this functionality, remember to uncomment the `require __DIR__ . '/bootstrap/media.php';` line in `app/config/bootstrap.php`.

At this point, our index view should be aware of the `$posts` `Document` object. Iterating through that object and printing out our post information is easy. Let's start over with our view in `/app/views/posts/index.html.php` with the following:
{{{
<?php foreach($posts as $post): ?>
<article>
	<h1><?=$post->title ?></h1>
	<p><?=$post->body ?></p>
</article>
<?php endforeach; ?>
}}}
As you can see, `Post` model objects expose their data through properties. Once this view has been saved, fire up your browser and check `/posts` to see the output.

## Post Mortem

There you have it: a fully functioning MVC system that performs writes and reads to a database. We hope this guide is illustrative enough to show you what Lithium can (and will be able to) do, and we hope it gives you enough of a start to allow you to start creating simple apps for yourself.

More fresh code is on the way, so stay tuned as more features come online.
