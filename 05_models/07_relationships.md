# Model Relationships

The data that applications manipulate is usually structured in some way. Objects have links to other objects, and the model layer that represents that data should reflect the structure inherent in the data it represents and interacts with.

li3's data layer offers a way to facilitate data relationships and structure. This guide is meant to show you how to specify and define these data relationships and use them to your advantage as you build your application.

## Defining Model Relationships

Before you define a model relationship in your code, it's important to understand the terminology that describes a relationship between two objects in your system. In li3 (and elsewhere), a specific set of terms are used to describe model relationships:

 * _hasOne_: the current object is linked to a single object of another type
 * _hasMany_: the current object is linked to many objects of another type
 * _belongsTo_: the current object is owned and marked as related to another object

<div class="note note-hint">
	If you're having a hard time remembering hasOne/hasMany versus belongsTo, just
	remember this: if the current model contains some sort of marker (like a foreign key), it
	<em>belongsTo</em> another model.
</div>

Defining this object relationship is simple: you populate special properties on the model object. For example, let's say we're building an online store. Each `Category` is filled with many `Product` objects. In this case, we'd want to specify `Category` hasMany `Product`. Let's see how this is done:

```php
class Categories extends \lithium\data\Model {

	public $hasMany = ['Products'];
}
```

This simple declaration relies on convention, and is the functional equivalent to this:

```php
class Categories extends \lithium\data\Model {

	public $hasMany = ['Products' => [
		'to'          => 'Products',
		'key'         => 'category_id',
		'constraints' => [],
		'fields'      => [],
		'order'       => null,
		'limit'       => null
	]];
}
```

Unless specified otherwise, the relationship assumes you're using the exact class name specified, with a key that is an under_scored version of the model's class name, suffixed with `_id`. All other sorting and limit options are assumed to be empty.

All of the model relationships use these same keys (although there's no reason to order or limit hasOne or belongsTo) and can be configured likewise.

## Reading Related Data

Once a relationship as been configured, you can use your models to fetch related data. We can now do this in a controller:

```php
$categories = Categories::find('all', [
	'with' => 'Products'
]);

print_r($categories->to('array'));
/* outputs:
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
```

Notice the new `with` key supplied to the model? This tells the model that you want related data joined to the normal result.

As you can see from the output, the related data has been added on to the model result. While we're printing out array contents here, you can as easily loop through or access the same information at `$categories->products` in this case as well.

## Ordering Related Data

Ordering data works mostly as you'd expect it to. The following returns
all categories and their associated products first unsorted and than sorted
by the category title.

Good practice is to qualify fields (with the model name) in such queries to make the
statement unambigous.

```php
Categories::find('all', [
	'with' => 'Products'
]);

Categories::find('all', [
	'with' => 'Products',
	'order' => ['Categories.title']
]);
```

To have the nested products themselves sorted inside the `->products`, you 
add the qualified relationship field to the order statement.

```php
Categories::find('all', [
	'order' => ['Categories.id', 'Products.price'],
	'with' => 'Products'
]);
```

Did you note that we also added `Categories.id` in there? This is to keep
a consistent result set. 

<div class="note note-caution">
	Sorting the whole result set just by the products 
	price alone is <em>not</em> possible.
</div>

If you see an exception thrown (`Associated records hydrated out of order.`), than 
simply order by the main models primary key first. Other frameworks will magically 
rewrite the query for you and add that primary key automatically. li3 however has
decided against this, as it doesn't want to mess with your queries.

## Saving Related Data

Because relationship setup is simple, so is saving related data. When saving related data, just make sure the proper key values are set so that the underlying data storage engine can match up the data correctly.
Here's a simplified example of how we'd save a newly created product, matched up to a category. First, the `ProductsController`:

```php
namespace app\controllers;

use \app\models\Products;
use \app\models\Categories;

class ProductsController extends \lithium\action\Controller {

	public function create() {
		// If form data has been submitted, create a new Product.
		if ($this->request->data) {
			Products::create($this->request->data)->save();
		}

		// Create a list of categories to send to the view.
		$categories = Categories::find('all');
		$categoryList = [];
		foreach ($categories as $category) {
			$categoryList[$category->id] = $category->name;
		}

		$this->set(compact('categoryList'));
	}
}
```

And, here is the view that contains the form:

```
<?= $this->form->create() ?>
	<?= $this->form->select('category_id', $categoryList) ?>
	<?= $this->form->text('name') ?>
	<?= $this->form->text('price') ?>
	<?= $this->form->submit('Create Product') ?>
<?= $this->form->end() ?>
```
