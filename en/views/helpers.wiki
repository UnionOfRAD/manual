# Helpers

Lithium helpers take the concept of elements a bit further, and create a place to house reusable presentation logic across your application. The framework comes with a few of it's own, and creating your own application-specific helpers is a snap.

## Usage

Helper usage in Lithium is simple because helpers are lazy-loaded by the renderer. To use a helper in the view layer, just reference it as a property of the renderer:

{{{<p>
	Here is a <?=$this->html->link('link', 'http://lithify.me') ?> you'll all enjoy.
</p>}}}

This approach means you won't have to declare which helpers you're planning on using. Ever. This also means that things aren't loaded up unless (and until!) you're actually using them.

Remember, you can use helpers anywhere in the view layer: inside layouts, view templates or elements.

## Creating Your Own Helpers

To create a custom helper, create a class in the `app/extensions/helper/` directory that extends Lithium's base `Helper` class. As a simple example, let's create a simple helper that creates a special sort of link by creating a new file in `app/extensions/helper/AwesomeHtml.php`:

{{{
<?php

namespace app\extensions\helper;

class AwesomeHtml extends \lithium\template\Helper {
  
	public function link($title, $url) {
		return "<a class=\"awesome\" href=\"$url\">$title</a>";
	}
}

?>
}}}

One important note to consider here is that the contents of `$title` and `$url` aren't escaped since they're being returned raw from the helper method. _Make sure you make liberal usage of the `escape()` method of the `Helper` class to keep things safe_.

Because string construction can get a little messy, you may wish to make use of the `_render()` method of the `Helper` class as well. This works when you create an array of string templates as the `_strings` property of a helper. Let's secure our example helper and make it a bit more flexible:

{{{
<?php

namespace app\extensions\helper;

class AwesomeHtml extends \lithium\template\Helper {

	protected $_strings = array(
		'awesome'    => '<a class="awesome" href="{:url}">{:title}</a>',
		'super_cool' => '<span class="super"><a class="cool" href="{:url}">{:title}</a></span>',
	);

	public function link($title, $url, $options) {
		$title = $this->escape($title);
		return $this->_render(__METHOD__, $options['type'], compact('title', 'url'));
	}
}

?>
}}}

Once this has been setup, we can use the new helper as we would any of the core helpers:

{{{<p>
	You should really check out
	<?=$this->awesomeHtml->link('Lithium', 'http://lithify.me', array(
		'type' => 'super_cool'
	)) ?>
</p>}}}

## Extending (and replacing) Core Helpers

Lithium gives preference to your classes over the core. If you want to create your own version of the core helpers, it's as easy as creating a new class inside the `app/extensions/helper` directory with the same name.

For example, we can replace Lithium's core `Html` helper class with our newly created helper class by renaming the class and filename to 'Html' rather than 'AwesomeHtml'. Doing this also allows you to leave your templates untouched: calls to `$this->html` will reference your helper automatically.

Also realize that in doing this, you'll probably wish to extend the helper you're replacing to take advantage of the functionality already there. If we took our own advice, the new helper would extend `lithium\template\helper\Html` and `link()` would be refactored to take advantage of the `Html` helper's existing link function.