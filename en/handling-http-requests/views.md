# Views

As one of the three pillars of the Model-View-Controller design pattern, the `View` class (along with other supporting classes) is responsible for taking the data passed from the request and/or controller, inserting this into the requested template/layout, and then returning the fully rendered content.

Unless otherwise specified, each controller action method results in a rendered view (usually as HTML). View names and locations are determined by convention, according the controller and action names involved. The basic pattern is that views are organized inside `app/views/{controller name}`, with each template named `{action name}.{media}.php`. For example:

 * `UsersController::login()` --> `/app/views/users/login.html.php`
 * `NewsItemsController::viewAll()` --> `/app/views/news_items/view_all.html.php`

## Accessing View Variables

Controller-layer data is made available to the template by means of view variables. The names of these variables and their contents depends on how they were passed along by the controller. As covered in the Controller guide, view variables are populated by the use of the `set()` method, or by returning associative arrays from your controller action methods.

Keys in those arrays determine the names of the view variables. The following lines of code placed in a controller action method all result in a view variable named `$foo` with the contents `"bar"`:

{{{
$this->set(array('foo' => 'bar'));

// -- or --

$foo = 'bar';
$this->set(compact('foo'));

// -- or --

return array('foo' => 'bar');

// -- or --

$foo = 'bar';
return compact('foo');
}}}

Once this has been done in the controller, you can access the data like so:

{{{<p>Spit out data like this: <?=$foo ?></p>}}}



Lithium templates are just PHP, so feel free to toss in conditionals, loops and other presentation-based logic as needed.

## Auto-Escaping

You might have noticed that the above example uses the short tag syntax to output the contents of a view variable. This syntax is a bit misleading, as Lithium does not depend on or use short tags: this output behavior works a bit differently from how it seems.

When the view layer is rendered, each template is processed by a tokenizer before it is compiled into its final form. During this step something like this:

{{{
<?=$variable; ?>
}}}

Is translated into something like this:

{{{
<?php echo $h($variable); ?>
}}}

The `$h()` function you see used there escapes HTML. To make a long story short, this mechanism provides an easy way for you to make sure all dynamically-generated data is safely landing in your HTML template.

We highly recommend using the `<?= ...; ?>` syntax in your views, as it aids greatly in hardening your application against cross-site scripting (and related) attack techniques.

 _Note:_ One exception to this rule is when a line of template code references the `$this` object. In those cases, output is written directly to the template, rather than being filtered through `$h()`. This is so that content from helpers is not double-escaped. As such, the following two statements are equivalent:

{{{
<?=$this->form->create(); ?>

<?php echo $this->form->create(); ?>
}}}

This is an important consideration when accessing properties and methods from the template renderer. If you intend to echo content directly from `$this` which is not coming from a helper (this is not a common occurence), you must manually escape it, like so:

{{{
<?php echo $h($this->foo); ?>
}}}

## Layouts & Elements

Because different components of the view layer are often reused, Lithium includes the common functionality of wrapping view templates inside of layouts and including small, re-usable view components called 'elements' inside of views. Unless otherwise configured, elements are defined in `app/views/elements`.

Layouts contain the header and footer of a rendered view, and are defined in `app/views/layouts`. Unless otherwise directed, Lithium will wrap a view's template with the layout defined in `app/views/layouts/default.{type}.php`. 

### Output Handlers

In the view rendering context, you'll have a number of different tools at your disposal. Besides PHP itself, the view layer features a number of handlers you can use to print out information in templates.

If you're in the view layer, `$this` refers to the current `Renderer` adapter. Renderers are used to process different types of templates. The default renderer that ships with Lithium is the `File` renderer, which handles plain PHP files (you can also write your own renderers if you wish to use a custom templating engine). When a renderer is initialized it sets up a number of handlers to help you output content in your views. Lithium's default view renderer features these handlers:

 * `$this->url()`: Used for reverse routing lookups in views. For example: 

 {{{
   $this->url(array('Posts::view', 'id' => 13));
   // Returns the URL for the matching route, e.g. '/posts/view/13'
 }}}

 The `url()` output handler is a friendly wrapper for [`Router::match()`](http://lithify.me/docs/lithium/net/http/Router::match), and automatically passes some contextual parameters behind-the-scenes.

 * `$this->path()`: This handler generates asset paths to physical files. This is especially helpful for applications that will live at different parts of the domain during its lifecycle.

 {{{
   $this->path('videos/funny_cats.flv');

   // If we're running at http://example.com/lithium/, this will return:
   // /lithium/videos/funny_cats.flv

   // If we're running at http://example.com/, this will return:
   // /videos/funny_cats.flv
 }}}
 Like `url()`, this handler is a wrapper for another class method, [`Media::asset()`](http://lithify.me/docs/lithium/net/http/Media::asset). Therefore, options that this method accepts can be passed to `path()` as well. For example, if you want to verify that the asset exists before returning the path, you can do the following:

 {{{
  // Returns the path if the file exists, otherwise false.
  $path = $this->path('files/download_835.pdf', array('check' => true));
 }}}

 You can also add a timestamp to the end of the URL, which is useful for working with browser caches:

 {{{
  // Returns i.e. '<app path>/css/application.css?1290108597'
  $style = $this->path('css/application.css', array('timestamp' => true));
 }}}

 See the `$options` parameter of `Media::asset()` for more information.

 * `$this->content()`: Prints out the content of the template to be rendered. This is really a requirement for most layouts. 

 * `$this->title()`: Prints out the title of the current template. If an argument is supplied, it sets the title for the current template.

 {{{
   // In your view:
   <?php $this->title('Home'); ?>

   // In your layout:
   <title>My Awesome Application: <?= $this->title(); ?></title>
 }}}

 * `$this->scripts()`: Prints out the scripts specified for the current template. Usually, this is used by the [`script()` method of the `Html` helper](http://lithify.me/docs/lithium/template/helper/Html::script) when the `'inline'` option is set to `false`. However, you can append tags manually as well:

 {{{
   // In your view:
   <?php 
    $this->scripts('<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.4.4/jquery.min.js"></script>'); 
    $this->scripts('<script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.8.7/jquery-ui.min.js"></script>');
   ?>

   // In your layout:
   <head>
     <?=$this->scripts(); ?>
   </head>
 }}}

 In particular, this is useful when page-specific scripts are created inline in the page, and you'd like to place them in the `<head />` along with your other scripts:

 {{{
  <?php ob_start(); ?>
  <script type="text/javascript">
  	$(document).ready(function() {
  		// Do work here...
  	});
  </script>
  <?php $this->scripts(ob_get_clean()); ?>
 }}}

 * `$this->styles()`: Much the same as `scripts()` above, the `styles()` handler acts as a repository for any page-specific style sheets to be included in the layout. While primarily used by the [`style()` method of the `Html` helper](http://lithify.me/docs/lithium/template/helper/Html::style) (again, see the `'inline'` option), it may be used manually as well:

 {{{
    // In your view:
    <?php
     $this->styles('<link rel="stylesheet" type="text/css" href="/reset.css" />');
     $this->styles('<link rel="stylesheet" type="text/css" href="/users.css" />');
    ?>

    // In your layout:
    <head>
      <?= $this->styles(); ?>
    </head>
  }}}
