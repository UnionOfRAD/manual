# Quickstart
## The Quintessential Blog Tutorial

Thanks for trying out li3! This section of the manual is for PHP users who want to see what the framework can do. Diving into the code like this is a great way to get a feel for Rapid Application Development!

By the time you have finished this section you will have built a simple blogging platform that reads and writes from a database. Later sections of the manual explain the concepts you'll use here in more detail.

## Setting Up

First things first: let's make sure our first project is installed and working. If you haven't already installed one then check out the [installation guide](./installation) in this manual. Make sure to follow each of the steps in the guide carefully.

After completing installation, you should be able to navigate your browser to the path that you installed your intitial project (e.g. `http://project.dev` or `http://127.0.0.1:8080`, the rest of this blog tutorial assumes you have installed it there) and it will display a page that starts like the snippet below. Note that some of the ticks and crosses may be different depending on what is set up on your system, but all of the boxes should be green or blue. If they aren't then please follow the instructions beneath them to fix the problem.

![Example initial li3 homepage](/assets/img/default_app_welcome.png "Example initial li3 homepage")

## MongoDB

We also need some sort of persistent storage layer for our blog posts. In this example, we're going to use [MongoDB](http://www.mongodb.org), a NoSQL database. One of the advantages of a NoSQL database is that you don't have to specify the schema upfront, so it works perfectly with our RAD approach to development!


<div class="note note-info">
	The framework has support for both relational (MySQL, MariaDB, PostgreSQL, SQLite) and NoSQL databases (MongoDB, CouchDB) and is one of the first frameworks to support both types of databases through one single ORM.
</div>

To initialize the database, follow the instructions for your operating system inside the _Installation_ chapter of the [official MongoDB manual](http://docs.mongodb.org/manual/) and finally, start up the database engine.

So that PHP can talk to Mongo you'll also need to install the PHP MongoDB driver. This is outlined
in the [official MongoDB manual](http://docs.mongodb.org/manual/) under [PHP Driver](http://docs.mongodb.org/ecosystem/drivers/php/). Once the driver is installed, we'll have to restart our webserver to make changes take effect.

Now, if you refresh the browser page for your installation, you should see a green tick by MongoDB like in the image above. If you  still have a cross or run into any other problems along the way, MongoDB's [Getting Started guide](http://docs.mongodb.org/manual/tutorial/getting-started/) is a great place to get some help.

Now that we have a database up and running and talking to PHP, let's tell li3 about it. Do this by editing `project/app/config/bootstrap/connections.php`. First, remove any connections that exist in the file, then add a new connection for our blog:

```php
// MongoDB Connection
Connections::add('default', [
	'type' =>  'MongoDb', 
	'database' => 'blog', 
	'host' => 'localhost'
]);
```

The first parameter just names the connection something that can be read by people. li3 also automatically uses the `'default'` connection elsewhere in our code unless otherwise specified.

The second array-type parameter is used to specify the connection. In this example, we're specifying a connection with `'type'` `'MongoDb'` for a database called `'blog'` on the `'localhost'` MongoDB server. These parameters can be specified in a number of different ways - see the `project/app/bootstrap/connections.php` file for more information.

<div class="note note-hint">
	Editing bootstrap files like this is a common way of configuring li3. See the <a href="./configuration/bootstrapping.md">Bootstrapping Guide</a> for more detail on how to configure the framework.
</div>

Our application is now set up and talking to the MongoDB database server, so we are ready to begin coding our blogging platform!

## MVC Starts with M

li3 uses the [MVC pattern](./architecture/mvc.md). If you're not familiar with using this pattern in web development you'll want to read up on it later, but for now let's create your first model, a `Posts` model that will handle the domain logic for blog posts.

First, create a new file at `project/app/models/Posts.php`. If you name your files and structure your code according to li3's conventions, the core library code will automatically do the heavy (and monotonous) lifting. This means that the model file itself is short and simple.

```php
namespace app\models;

class Posts extends \lithium\data\Model {}
```

There's a lot going on (for free!) in the background here. First, li3 knows we're using our default connection because we haven't specified otherwise. Secondly, since our model is named `Posts`, it'll use a MongoDB collection called 'posts'. We've also inherited a lot of useful methods for querying the database, and we'll learn more about some of those in later sections of this tutorial.

But wait, what? What about the schema setup? Actually, MongoDB doesn't require you to set it up first - it just waits until you want to insert or query the database and handles the request appropriately. That's RAD!

## In Control

The controller setup is just as simple when getting started. Create a new file at `project/app/controllers/PostsController.php` and fill it with the following:

```php
namespace app\controllers;

class PostsController extends \lithium\action\Controller {}
```

You may have noticed a trend: filenames are CamelCase, as are classnames. Folder paths match their respective namespace, and are under_scored. This is part of the li3 [coding convention](/docs/specs/accepted/LSR-0-coding.md), and you should stick to it to take full advantage of the framework's automagic.

Let's go ahead and create an initial action as well. Create a new `index()` function in your newly created controller. Before we try and link all three of the model, controller and viewer together, let's just set up a simple action that pushes some dummy data to the view. Here's how it's done:

```php
namespace app\controllers;

class PostsController extends \lithium\action\Controller {

	public function index() {
		return ['foo' => 'bar', 'title' => 'Posts'];
	}
}
```

li3 actions send data to the view by returning an associative array that determines the respective view's variables. In the example above, our view will have access to the string `'bar'` in `$foo` and `'Posts'` in `$title` (`$title` will be used to set the page's title tag). This setup is also handy for fans of the [`compact()` function](http://php.net/compact). If I have a series of variables I'd like to pass along, it's as easy as:

```php
return compact('lions', 'tigers', 'bears');
```

## What a View

Next, let's create a `View` that uses the dummy data from `PostsController::index()`.

Start by creating a new file at `project/app/views/posts/index.html.php` (you'll need to create the posts directory). Let's start simple, and print out the data we so carefully crafted in our controller:

```
li3 is less dense than <?=$foo;?>ium.
```

Save this file, and now you can view the Posts index page by pointing your browser to `http://project.dev/posts`. li3 handles the routing and dispatching behind the scenes. You can learn more about setting up custom routes in the [controller section](../controllers/routing.md) of the manual.

What you're seeing used here in this view's code is also the default and preferred way to output data to an HTML page in li3. The short tags (`<?= ... ?>`) are automatically rewritten by li3 to escape the output and keep you safe from the legions of attacks based on unescaped output. This means that the view code you're seeing doesn't end up as short tags when it gets to PHP's parser, so don't worry about your PHP installation or short tag handling. If you really need it, there's also always `<?php echo ... ?>`.

## First Post!

Now that you've built a li3 application that displays some dummy data, it's time to extend it so that users can write posts.

First, create a new action in `PostsController` called `add()`. 

```php
namespace app\controllers;

use app\models\Posts;

class PostsController extends \lithium\action\Controller {

	public function add() {
		$post = Posts::create();
		return compact('post');	
	}
}
```

Before you can use a model and its methods we must tell that we're planning to use the moodel class via the `use` statement. The statement needs to be added at the beginning of our `PostsController.php` file, but _after_ the namespace declaration.

Currently - inside the action - we just create an empty post object and pass it to the view which we'll create next in a new file at `project/app/views/posts/add.html.php`:

```
<?=$this->form->create($post); ?>
	<?=$this->form->field('title');?>
	<?=$this->form->field('body', ['type' => 'textarea']);?>
	<?=$this->form->submit('save'); ?>
<?=$this->form->end(); ?>
```

This view code sets up a simple HTML form, using the `Form` helper. Don't stress the details of what the helper is doing at this point - what it outputs is most important for now.

When we call `$this->form->create()` we pass the currently empty post object as the first argument. This binds the post to the form we're about to create and allows the `Form` helper to _know_ about the post schema. Note that we don't include an `url` parameter when creating the form. li3 assumes you mean the controller method that corresponds to the name of the view (in this case `add()`) and want to use POST. This is another example of how li3 defaults to the most common case so that your code can be as concise and expressive as possible - in this case, it's pointed to `http://project.dev/posts/add`, just as we need.

Let's move back to the controller, and handle the data the HTML form is sending us. Here's what our `add()` action should now look like:

```php
namespace app\controllers;

use app\models\Posts;

class PostsController extends \lithium\action\Controller {

	public function add() {
		$post = Posts::create();
		$success = false;

		if ($this->request->data && $post->save($this->request->data)) {
			$success = true;
		}
		return compact('post', 'success');
	}
}
```

Because the form we've created in the view submits its data back to the same controller action, all controller code for adding a new post happens in this method. 

First, the method checks to see if there's any POST data in the response. In this case, the data is passed into `save()` which updates the empty post object and saves it at the same time.

The result of the save operation (`$success`) is then passed back to the view so we can display a message to the user, and we can use that data in the view to show some sort of status message:

```
<?php if ($success): ?>
	<p>Post Successfully Saved</p>
<?php endif; ?>
```

To give you a little more of an idea what li3's doing "behind the scenes" here, the following pieces of code are equivalent:

```php
$post = Posts::create(['title' => 'First Post', 'body' => 'Body text!']);
$post->save();

// or:

$post = Posts::create();
$post->title = 'First Post';
$post->body  = 'Body Text!';
$post->save();

// or: 

$post = Posts::create();
$post->save(['title' => 'First Post', 'body' => 'Body text!']);
```

Feel free now to have a play with this method and use your browser to add a few posts to your blog. Coming up next: displaying them!

## Viewing your posts

Now that we've stocked our database, let's use our Posts model to pull the data. We can do this by rewriting the `index()` action in the `PostsController` to pull a list of posts and hand it to the view:

```php
public function index() {
	$posts = Posts::all();
	return compact('posts');
}
```

The object returned from `Posts:all()` - which is equivalent to `Posts::find('all')` - is a li3 `Document` object. The `Document` object is a response object that handles MongoDB responses more appropriately, since it's not a traditional RDBMS. Don't worry too much about this now: just understand that what you're getting back isn't plain array-based data.

If for some reason you need to inspect the data (for debugging purposes, for example), you can use the `to()` method on the document object.

```php
var_dump($posts->to('array'));
// example output:
//
// [0]=>
//  [3] {
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
<?php foreach ($posts as $post): ?>
<article>
	<h1><?= $post->title ?></h1>
	<p><?= $post->body ?></p>
</article>
<?php endforeach; ?>
```

As you can see, `Posts` model `Document` objects expose their data through properties. Once this view has been saved, fire up your browser and check `http://project.dev/posts` to see the output.

