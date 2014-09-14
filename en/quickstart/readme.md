# The Quintessential Blog Tutorial

Thanks for trying out Lithium! This section of the manual is for PHP
users who want to see what Lithium can do. Diving into the code like this is a great way to get a feel for Rapid Application Development!
By the time you have finished this section you will have built a simple blogging platform that reads and writes from a database. Later sections of the manual explain the concepts you'll use here in more detail.

## Setting Up Lithium

First things first: let's make sure Lithium is installed and working. If you haven't already installed Lithium then check out the [installation guide](../installation/installation.md) in this manual. Make sure to follow each of the steps in the guide carefully.

After completing installation, you should be able to navigate your browser to the path that you installed Lithium to (e.g. `my_app`, the rest of this blog tutorial assumes you have installed Lithium here) and it will display a page that starts like the snippet below. Note that some of the ticks and crosses may be different depending on what is set up on your system, but all of the boxes should be green or blue. If they aren't then please follow the instructions beneath them to fix the problem.

----

![Example initial Lithium homepage](http://i.imgur.com/MsGsjel.png "Example initial Lithium homepage")
----

## MongoDB

As well as Lithium itself, we also need some sort of persistent storage layer for our blog posts. In this example, we're going to use MongoDB (http://www.mongodb.org), a NoSQL database. One of the advantages of a NoSQL database is that you don't have to specify the schema upfront, so it works perfectly with Lithium's RAD approach to development!

The easiest way to get started is by downloading a binary from the MongoDB website. Check their [downloads page](http://www.mongodb.org/display/DOCS/Downloads), and fetch the binary that corresponds to your platform.

Next, perform a bit of setup by creating a data directory for Mongo. Do this by creating a `/data/db` directory on your system (`C:\data\db` for our Windows friends).

Finally, start up the database engine. Run `/path/to/mongodb/bin/mongod`, and wait for the initialization information to finish showing.

## PHP and Mongo

So that PHP can talk to Mongo you'll also need to install the Mongo PECL module. For most systems ([not Windows](http://www.mongodb.org/display/DOCS/Installing+the+PHP+Driver#InstallingthePHPDriver-WindowsInstall)), this is as easy as:

```sh
sudo pecl install mongo
```

Once that's finished successfully, make sure to add `extension=mongo.so` to your `php.ini` file and restart the server, e.g.:

```sh
sudo /etc/init.d/apache2 restart
```

The `php.ini` file will be stored in a directory like `/etc/php5/apache2` (exactly where depends on your system and server). There may also be a `php.ini` file for CLI. We don't use the `li3` command line application in this quickstart, but you might as well add `extension=mongo.so` to this other `php.ini` file as well to enable the command line application for the future.

Now, if you refresh the browser page for your installation, you should see a green tick by MongoDB like in the image above. If you  still have a cross or run into any other problems along the way, MongoDB's [Getting Started guide](http://www.mongodb.org/display/DOCS/Getting+Started) is a great place to get some help.

## Connection Setup

Now that we have a database up and running and talking to PHP, let's tell Lithium about it. Do this by editing `my_app/app/config/bootstrap/connections.php`. First, remove any connections that exist in the file, then add a new connection for our blog:

```php
// MongoDB Connection
Connections::add('default', array('type' =>  'MongoDb', 'database' => 'blog', 'host' => 'localhost'));
```

The first parameter just names the connection something that can be read by people. Lithium also automatically uses the `'default'` connection elsewhere in our code unless otherwise specified.

The second array-type parameter is used to specify the connection. In this example, we're specifying a connection with `'type'` `'MongoDb' for a database called `'blog'` on the `'localhost'` MongoDB server. These parameters can be specified in a number of different ways - see the [API documentation](http://lithify.me/docs/app/config/bootstrap/connections) for more information.

Editing bootstrap files like this is a common way of configuring Lithium. For example, you might want to set up some quick and dirty error handling by adding `ini_set("display_errors", 1);` to `my_app/app/config/bootstrap.php` - see [this section](../configuration/bootstrapping.md) of the manual for more detail on how to configure Lithium. Lithium is now set up and talking to the MongoDB database server, so we are ready to begin coding our blogging platform!

## MVC Starts with M

Lithium uses the [MVC pattern](../design-principles/mvc.md). If you're not familiar with using this pattern in web development you'll want to read up on it later, but for now let's create your first Lithium model, a `Posts` model that will handle the domain logic for blog posts.

First, create a new file at `my_app/app/models/Posts.php`. If you name your files and structure your code according to Lithium's conventions, the core library code will automatically do the heavy (and monotonous) lifting. This means that the model file itself is short and simple.

```php
namespace app\models;

class Posts extends \lithium\data\Model {}
```

There's a lot going on (for free!) in the background here. First, Lithium knows we're using our default connection because we haven't specified otherwise. Secondly, since our model is named `Posts`, it'll use a MongoDB collection called 'posts'. We've also inherited a lot of useful methods for querying the database, and we'll learn more about some of those in later sections of this tutorial.

But wait, what? What about the schema setup? Actually, MongoDB doesn't require you to set it up first - it just waits until you want to insert or query the database and handles the request appropriately. That's RAD!



## In Control

The controller setup is just as simple when getting started. Create a new file at `my_app/app/controllers/PostsController.php` and fill it with the following:

```php
namespace app\controllers;

class PostsController extends \lithium\action\Controller {}
```

You may have noticed a trend: filenames are CamelCase, as are classnames. Folder paths match their respective namespace, and are under_scored. This is part of the Lithium [coding convention](../quality-code/coding-standards.md), and you should stick to it to take full advantage of the framework's automagic.

Let's go ahead and create an initial action as well. Create a new `index()` function in your newly created controller. Before we try and link all three of the model, controller and viewer together, let's just set up a simple action that pushes some dummy data to the view. Here's how it's done:

```php
namespace app\controllers;

class PostsController extends \lithium\action\Controller {

	public function index() {
		return array('foo' => 'bar', 'title' => 'Posts');
	}
}
```

Lithium actions send data to the view by returning an associative array that determines the respective view's variables. In the example above, our view will have access to the string `'bar'` in `$foo` and `'Posts'` in `$title` (`$title` will be used to set the page's title tag). This setup is also handy for fans of the [`compact()` function](http://php.net/compact). If I have a series of variables I'd like to pass along, it's as easy as:

```php
return compact('lions', 'tigers', 'bears');
```

## What a View

Next, let's create a `View` that uses the dummy data from `PostsController::index()`.

Start by creating a new file at `my_app/app/views/posts/index.html.php` (you'll need to create the posts directory). Let's start simple, and print out the data we so carefully crafted in our controller:

```
Lithium is less dense than <?=$foo;?>ium.
```

Save this file, and now you can view the Posts index page by pointing your browser to `my_app/posts`. Lithium handles the routing and dispatching behind the scenes. You can learn more about setting up custom routes in the [controller section](../controllers/routing.md) of the manual.

What you're seeing used here in this view's code is also the default and preferred way to output data to an HTML page in Lithium. The short tags (`<?= ... ?>`) are automatically rewritten by Lithium to escape the output and keep you safe from the legions of attacks based on unescaped output. This means that the view code you're seeing doesn't end up as short tags when it gets to PHP's parser, so don't worry about your PHP installation or short tag handling. If you really need it, there's also always `<?php echo ... ?>`.



## First Post!

Now that you've built a Lithium application that displays some dummy data, it's time to extend it so that users can write posts.

First, create a new action in `PostsController` called `add()`:

Next, create a new file at `my_app/app/views/posts/add.html.php`:

```
<?=$this->form->create(); ?>
	<?=$this->form->field('title');?>
	<?=$this->form->field('body', array('type' => 'textarea'));?>
	<?=$this->form->submit('Add Post'); ?>
<?=$this->form->end(); ?>
```

This view code sets up a simple HTML form, using a view helper class called FormHelper. Don't stress the details of what the helper is doing at this point - what it outputs is most important for now.

Note that because the call to `$this->form->create()` doesn't include any parameter, Lithium assumes you mean the controller method that corresponds to the name of the view (in this case `add()`) and want to use POST. This is another example of how Lithium defaults to the most common case so that your code can be as concise and expressive as possible - in this case, it's pointed to `my_app/posts/add`, just as we need.

Let's move back to the controller, and handle the data the HTML form is sending us. Here's what our `add()` action should now look like:

```php
public function add() {
	$success = false;
	if ($this->request->data) {
		$post = Posts::create($this->request->data);
		$success = $post->save();
	}
	return compact('success');
}
```

Because the form we've created in the view submits its data back to the same controller action, all controller code for adding a new Post happens in this method. First, the method checks to see if there's any POST data in the response. In this case, it would be filled with our form data so we `create()` a new Post model with the request data and save it to the database with `save()`. The result of any save operation (`$success`) is then passed back to the view so we could display a message to the user, and we could use that data in the view to show some sort of status message like so:

```
<?php if ($success): ?>
	<p>Post Successfully Saved</p>
<?php endif; ?>
```

To give you a little more of an idea what Lithium's doing "behind the scenes" here, the following pieces of code are equivalent:

```php
$post = Posts::create(array('title' => 'First Post', 'body' => 'Body text!'));

// or:

$post = Posts::create();
$post->title = 'First Post';
$post->body = 'Body Text!';
```

If you run this code right now though it won't work and will generate an error message. That's because the controller knows nothing about the model and its `create()` or `save()` methods, so we must tell it! At the beginning of our `PostsController.php` file, but _after_ the namespace declaration, we add a `use app\models\Posts;` line:

```php
namespace app\controllers;

use app\models\Posts;

class PostsController extends \lithium\action\Controller {
	// ...
}
```

Feel free now to have a play with this method and use your browser to add a few posts to your blog. Coming up next: displaying them!



## Viewing your posts

Now that we've stocked our database, let's use our Post model to pull the data. We can do this by rewriting the `index()` action in the `PostsController` to pull a list of posts and hand it to the view:

```php
public function index() {
	$posts = Posts::all();
	return compact('posts');
}
```

The object returned from `Post:all()` - which is equivalent to `Posts::find('all')` - is a Lithium `Document` object. The `Document` object is a response object that handles MongoDB responses more appropriately, since it's not a traditional RDBMS. Don't worry too much about this now: just understand that what you're getting back isn't plain array-based data.

If for some reason you need to inspect the data (for debugging purposes, for example), you can use the `to()` method on the document object.

```php
var_dump($posts->to('array'));
// example output:
//
// [0]=>
//  array(3) {
//    ["_id"]=>
//    string(24) "4afddab78bd3c34f41757396"
//    ["title"]=>
//    string(11) "First Post!"
//    ["body"]=>
//    string(7) "Woooooo"
//  }
```

If you're curious, you might also try `$posts->to('json')` to see some additional options. To enable this functionality, remember to uncomment the `require __DIR__ . '/bootstrap/media.php';` line in `app/config/bootstrap.php`.

At this point, our index view should be aware of the `$posts` `Document` object. Iterating through that object and printing out our post information is easy. Replace the existing code in `/app/views/posts/index.html.php` with the following:

```
<?php foreach($posts as $post): ?>
<article>
	<h1><?= $post->title ?></h1>
	<p><?= $post->body ?></p>
</article>
<?php endforeach; ?>
```

As you can see, `Post` model `Document` objects expose their data through properties. Once this view has been saved, fire up your browser and check `my_app/posts` to see the output.

