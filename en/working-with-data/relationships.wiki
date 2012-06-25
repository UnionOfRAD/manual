# Model Relationships

The data that applications manipulate is usually structured in some way. Objects have links to other objects, and the model layer that represents that data should reflect the structure inherent in the data it represents and interacts with.

Lithium's data layer offers a way to facilitate data relationships and structure. This guide is meant to show you how to specify and define these data relationships and use them to your advantage as you build your application.

## Defining Model Relationships

Before you define a model relationship in your code, it's important to understand the terminology that describes a relationship between two objects in your system. In Lithium (and elsewhere), a specific set of terms are used to describe model relationships:

 * hasOne: the current object is linked to a single object of another type
 * hasMany: the current object is linked to many objects of another type
 * belongsTo: the current object is owned and marked as related to another object

If you're having a hard time remembering hasOne/hasMany versus belongsTo, just remember this: if the current model contains some sort of marker (like a foreign key), it _belongsTo_ another model.

Defining this object relationship in Lithium is simple: you populate special properties on the model object. For example, let's say we're building an online store. Each `Category` is filled with many `Product` objects. In this case, we'd want to specify `Category` hasMany `Product`. Let's see how this is done:

{{{
<?php
class Categories extends \lithium\data\Model {
	public $hasMany = array('Products');
}
?>
}}}

This simple declaration relies on convention, and is the functional equivalent to this:

{{{
<?php
class Categories extends \lithium\data\Model {
	public $hasMany = array('Products' => array(
		'to'          => 'Products',
		'key'         => 'category_id',
		'constraints' => array(),
		'fields'      => array(),
		'order'       => null,
		'limit'       => null
	));
}
?>
}}}

Unless specified otherwise, the relationship assumes you're using the exact classname specified, with a key that is an under_scored version of the model's class name, suffixed with `_id`. All other sorting and limit options are assumed to be empty.

All of Lithium's model relationships use these same keys (although there's no reason to order or limit hasOne or belongsTo) and can be configured likewise.

## Reading Related Data

Once a relationship as been configured, you can use your models to fetch related data. We can now do this in a controller:

{{{
<?php

$categories = Categories::find('all', array(
	'with' => 'Products'
));

/*
	Output of $categories->to('array'):

	Array
	(
	    [id] => 1
	    [name] => Audio
	    [products] => Array
	        (
	            [0] => Array
	                (
	                    [id] => 1
	                    [category_id] => 1
	                    [name] => Headphones
	                    [price] => 21.99
	                )
	            [1] => Array
	                (
	                    [id] => 2
	                    [category_id] => 1
	                    [name] => Desk Speakers
	                    [price] => 39.95
	                )
	            [2] => Array
	                (
	                    [id] => 3
	                    [category_id] => 1
	                    [name] => Portable Radio
	                    [price] => 9.95
	                )
	        )
	)
*/
?>
}}}

Notice the new `with` key supplied to the model? This tells Lithium that you want related data joined to the normal response.

As you can see from the output, the related data has been added on to the model response. While we're printing out array contents here, you can as easily loop through or access the same information at `$categories->products` in this case as well.

## Saving Related Data

Because Lithium's relationship setup is simple, so is saving related data. When saving related data, just make sure the proper key values are set so that the underlying data storage engine can match up the data correctly.

Here's a simplified example of how we'd save a newly created product, matched up to a category. First, the `ProductsController`:

{{{
<?php
namespace app\controllers;

use \app\models\Products;
use \app\models\Categories;

class ProductsController extends \lithium\action\Controller {
	public function create() {
		// If form data has been submitted, create a new Product.
		if($this->request->data) {
			Products::create($this->request->data)->save();
		}
		// Create a list of categories to send to the view.
		$categories = Categories::find('all');
		$categoryList = array();
		foreach($categories as $category) {
			$categoryList[$category->id] = $category->name;
		}

		$this->set(compact('categoryList'));
	}
}
?>
}}}

And, here is the view that contains the form:

{{{
<?= $this->form->create() ?>
	<?= $this->form->select('category_id', $categoryList) ?>
	<?= $this->form->text('name'); ?>
	<?= $this->form->text('price'); ?>
	<?= $this->form->submit('Create Product') ?>
<?= $this->form->end(); ?>
}}}