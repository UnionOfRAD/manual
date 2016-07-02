# Elements

Since different components of the view layer are often reused, li3 includes the common functionality of wrapping view templates inside of layouts and including small, re-usable view components called <em>elements</em> inside of views.  You can also think of an element like a widget or other portion of the displayed content that would be reused across multiple pages.

### Examples of Elements

- site navigation
- single post items
- widgets i.e. upcoming events, blogroll.

Unless otherwise configured, elements are defined in `app/views/elements`. Inside this folder, the files you define will be tailored to display the output for your chosen element.  As an example, the code below defines a product element.  The file name is `app/views/elements/product.html.php`.

```html
<article class="product">
	<h1><?= $item->title ?></h1>
	<img src="<?= $item->cover ?>" />

	<div class="description">
		<?php echo $item->description ?>
	</div>
	<div class="price">
		$<?= $item->price_usd ?>
	</div>
</article>
```

### Displaying an Element

Displaying an element is accomplished via the `View::render()` method. In the following example we render a list of products using the product element we defined above.

```html
<div class="products">
<?php foreach ($products as $item): ?>
   <?php echo $this->_view->render(
	   ['element' => 'product'], compact('item')
   ) ?>
<?php endforeach ?>
</div>
```

A complete description of the `render()` method can be found in the li3 API documentation under [View::render()](/docs/api/lithium/latest:1.x/lithium/template/View::render)
