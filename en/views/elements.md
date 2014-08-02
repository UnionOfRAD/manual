# Elements

Since different components of the view layer are often reused, Lithium includes the common functionality of wrapping view templates inside of layouts and including small, re-usable view components called 'elements' inside of views.  You can also think of an element like a widget or other portion of the displayed content that would be reused across multiple pages.

###Examples of Elements###
- Site Navigation
- List of Recent Posts
- Upcoming Events
- Latest Social Media Activity

Unless otherwise configured, elements are defined in `app/views/elements`. Inside this folder, the files you define will be tailored to display the output for your chosen element.  As an example, the code below defines the navigation element for Lithium.  The file name is `app/views/elements/nav.html.php`.

``` <?php
if (!isset($object) || !$object) {
	return;
}
?>
<div class="aside crumbs">
	<aside>
		<h3>docs for</h3>
		<ul>
				<!-- <li class="home">
					<?=$this->html->link($t('\\', array('scope' => 'li3_docs')), array(
						'controller' => 'li3_docs.ApiBrowser', 'action' => 'index'
					), array('escape' => false)); ?>
				</li> -->
			<?php foreach ($this->docs->crumbs($object) as $crumb): ?>
				<li class="<?= $crumb['class']; ?>">
					<?php
						if ($crumb['url']) {
							echo $this->html->link($crumb['title'], $crumb['url']);
							continue;
						}
					?>
					<span><?=$crumb['title']; ?></span>
				</li>
			<?php endforeach; ?>
		</ul>
	</aside>
</div>```

###Displaying an Element###
Displaying an element is accomplished via the `View::render()` method.  In the following example, the default layout for Lithium uses the navigation element, with the element being displayed by using the following code:

```<div class="nav">
	   <nav>
		   <?php echo $this->_view->render(
			   array('element' => 'nav'), compact('object'), array('library' => 'li3_docs')
		   ); ?>
	   </nav>
   </div>```

A complete description of the `render()` method can be found in the Lithium API documentation under [View::render()](http://lithify.me/docs/lithium/template/View::render)