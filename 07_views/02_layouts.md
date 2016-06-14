# Layouts

A layout is the basic skeleton of an HTML page.  Within a layout file, you might typically have a header, footer, and any other HTML code that should appear on every page that uses the layout.

For layouts, all files should be located in the application's `views/layouts`  directory.  When naming an HTML layout file, the naming convention is `{layout_name}.html.php`.

As an example, the default template in li3 would be found at `views/layouts/default.html.php`.  You can also have layouts for other formats, such as XML, and you would name the file accordingly (e.g. `default.xml.php`).

A layout file looks like what you might typically expect of an HTML file.  Below is the code for the default template that ships with li3:

```
<!doctype html>
   <html>
   <head>
   	<?=$this->html->charset();?>
   	<title>Application > <?=$this->title();?></title>
   	<?=$this->html->style(array('debug', 'lithium'));?>
   	<?=$this->scripts();?>
   	<?=$this->html->link('Icon', null, array('type' => 'icon'));?>
   </head>
   <body class="app">
   	<div id="container">
   		<div id="header">
   			<h1>Application</h1>
   			<h2>
   				Powered by <?=$this->html->link('li3', 'http://li3.me/');?>.
   			</h2>
   		</div>
   		<div id="content">
   			<?=$this->content();?>
   		</div>
   	</div>
   </body>
   </html>
```

Within the layout file, there are several familiar components.  For example, in the <head> section, you find li3's code for displaying titles, linking CSS files & JS files, setting the Charset, etc.  There is a page header, and also the closing tags for body and html round out the layout file's code.

When it comes to displaying the content of the page, just include a statement to echo $this->content() in the layout wherever the content is going to be displayed.

##Using a Layout
Once you have created a layout, it is just a matter of applying the layout to your views when they are rendered from the controller.  This can be done from each controller method when the `render()` method is called as shown in the example below:

```
public function index() {
	return $this->render(array('layout' => 'layoutName'));
}
```

If you do not specify a layout in the render call, li3 will use the default layout, which is found in `views/layouts/default.html.php`.  You can also disable the layout altogether by setting the layout's value to false (e.g. `$this->render(array('layout' => false))`).

##Setting a Default Layout
You can also specify a default layout to be used for all methods in a controller.  This is done through the use of the protected `init()` method.  An example of how to set the default layout is shown below:

```
protected function _init() {
	parent::_init();
	$this->_render['layout'] = 'default';
}
```
