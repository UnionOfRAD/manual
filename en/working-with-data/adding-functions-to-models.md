# Adding Functionality to Lithium Models

This section outlines a few of the ways you can extend model functionality as you build your application. As you think about different pieces of functionality to add, please keep in mind that there are two main ways to add functionality to your models:

 1. Static methods that apply to the Model
 2. Instance methods that apply to each Entity (Document or Record)

If you remember this simple rule, you'll understand how the framework reacts to new functions placed in models—and you'll be able to use them more effectively.

## Static Data Access

One simple example is if you have a bit of data that is specific to a model's domain, and you want to make that data available to controllers using those models.

{{{
<?php
namespace app\models;

class Users extends \lithium\data\Model {
	protected static $_roles = array('admin', 'friend', 'stranger');

	public static function roles() {
		return static::$_roles;
	}
}
?>
}}}

Because this method is accessed statically, it behaves as you'd expect:

{{{
<?php
if(!in_array('admin', Users::roles())) {
	return false;
}
?>
}}}

## Adding Model "Finder" Methods

Lithium models ship with a number of default "finder" methods:

{{{
<?php
$users      = Users::find('all');
$oneUser    = Users::find('first');
$keyedArray = Users::find('list');
?>
}}}

As you use your models, you might start to wish for a shortcut. For example, instead of having to do this repeatedly:

{{{
<?php

$recentComments = Comments::find('all', array(
	'conditions' => array(
		'created_on' => array(
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		)
	)
));

?>
}}}

You could create a custom finder method that packages the specified conditions into a one-liner:

{{{
<?php
$recentComments = Comments::find('recent');

// or, as a "magic" method:

$recentComments = Comments::recent();
?>
}}}

At a basic level, this is done by utilizing the `finder()` method of the model. You call `finder()` and supply the name of the finder, along with a definition so Lithium knows how to form the query. The definition in this simple case looks just like the query array we supplied to `find()` earlier:

{{{
<?php
Comments::finder('recent', array(
	'conditions' => array(
		'created_on' => array(
			'>=' => date('Y-m-d H:i:s', time() - (86400 * 3))
		)
	)
));
?>
}}}

Some finder implementations might require a little processing in addition to a default set of conditions—somewhat like the . In that case, you can define a finder using a closure that will be called as part of Lithium's find chaining. In this use case, you supply the name of the finder along with a closure that looks much like a filter definition:

{{{
<?php 
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
	foreach($chain->next($self, $params, $chain) as $entity) {
		$results[] = $entity->categoryName;
	}
	
	// Returns an array of recent categories given the supplied query params.
	return $results;
});
?>
}}}

## Model Instance Methods

It's often useful to add a method to a model so that you can easily transform data once you've got a model instance (of type `Entity`, either `Document` or `Record`) in your controllers and views. In this case, you'll need to create a method that accepts the entity itself as the first argument, then any addition parameters (if any). Here's a simple use case of creating a method on a `Users` model that formats a full name based on first and last names:

{{{
<?php
namespace app\models;

class Users extends \lithium\data\Model {
	public function fullName($entity) {
		return $entity->firstName . ' ' . $entity->middleInitial . '. ' . $entity->lastName;
	}
}
?>
}}}

If you want to add additional parameters, do so after you've specified the entity as the first:

{{{
<?php
namespace app\models;

class Users extends \lithium\data\Model {
	public function fullName($entity, $suffix = false) {
		$name = $entity->firstName . ' ' . $entity->middleInitial . '. ' . $entity->lastName;
		if($suffix) {
			$name .= ' ' . $entity->suffix;
		}
		return $name;
	}
}
?>
}}}

Once this is done, use it wherever you've got access to a `Users` model instance:

{{{
<?php

$firstUser = Users::first();

$firstUser->fullName();  // "Bill S. Preston"
$firstUser->fullName(true);  // "Bill S. Preston Esq."

?>
}}}