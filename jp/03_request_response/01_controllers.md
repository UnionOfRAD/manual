# Controllers

In an MVC application, the controller's job is to handle user input, and handle that request with a specified action. Though the controller action is where action logic is determined, models are often the real workhorses in the request cycle. Keep in mind as you read this guide to keep your controller action trim, with most of the actual functionality spread across models and other classes.

This guide is mean to introduce you to li3 controllers: their features, behaviors, and the best practices revolving around them.

## Controller Actions

li3 controllers reside inside the `/app/controllers` directory and extend the `\lithium\action\Controller` core class. Let's start by creating a simple controller inside of an application. Controllers are often named after the object they manage. This way the URL and model line up as well, and it's easy to know where certain bits of logic should live. 

For example, let's create a new controller GalaxiesController (who doesn't wan't to control their own galaxy, after all). Let's create a new file in `/app/controllers/GalaxiesController.php` that looks like this:

```
<?php

namespace app\controllers;

class GalaxiesController extends \lithium\action\Controller { 
	public function index() {
		
	}
}
?>
```

Each _public_ function in a controller is considered by the li3 core to be a routable action. In fact, li3's default routing rules make these actions accessible via a browser immediately (in this case /galaxies/index). 

The `index()` action is a special action: if no action name is specified in the URL, li3 will try to pull up the index action instead. For example, a visitor accessing http://example.com/galaxies/ on your application will see the results of the `index()` action. All other controller actions (unless routed otherwise) are at least accessed by the default route, which is by action name.

Let's create our own action as an example:

```
<?php

namespace app\controllers;

class GalaxiesController extends \lithium\action\Controller { 
	public function index() {
		
	}
	
	public function eliminateResistance() {
	
	}
}
?>
```

## Accessing Request Parameters

## Request Flow Control

## Using Models 

## Moving Data to the View

## Using Core Libraries (Session, Service, etc)

## Filtering Controller Logic