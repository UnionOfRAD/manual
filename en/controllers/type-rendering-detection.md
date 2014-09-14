# Type Rendering and Detection
Although a typical request to a Lithium application receives an HTML response, the framework is built to be extremely flexible in handling and serving different types of content. This functionality is especially important in applications that have many different components or endpoints. If your app also feeds data to a Flash object (AMF/XML) and a mobile phone (XML/JSON), responding to requests in different ways with the same underlying logic can be a huge time saver.

The flow for handling a given type of a response works something like the following:

 1. A request is sent to the application, containing some sort of indicator of the request type. Lithium's default routing allows for simple extension detection, for example.
 2. As Lithium bootstraps, a media type and handler is registered with the `\net\http\Media` class.
 3. The application detects the request type and sets the response type.
 4. Once a controller is ready to render the data, the registered handler receives the data and renders the output.

## Detecting and Setting Types

The easiest way to set a type is by declaring it as part of the route. One of Lithium's default routes already does this for you:

```
Router::connect('/{:controller}/{:action}/{:id:[0-9]+}.{:type}', array('id' => null));
```

> NOTE:

> You will need to uncomment the appropriate default route in the application folder's `/config/routes.php` file in order to use the features described below.  There are preformatted routes for both document type data sources and relational database sources available in the `routes.php` file.

In effect, this forces a request to `/controller/action/7345.json` to be rendered by the JSON media handler currently registered. You can use this pattern to expand your routes to apply type-matching to a wider array of requests:

```
// http://example.com/controller/action.xml
Router::connect('/{:controller}/{:action}.{:type}');
```

### Example
Let's assume that you are using a MySQL database as a datasource and you are retrieving data from the `blog_posts` table.  First, we need to enable the routes in the `routes.php` file as shown below:

```
/**
 * ### Database object routes
 *
 * The routes below are used primarily for accessing database objects, where `{:id}` corresponds to
 * the primary key of the database object, and can be accessed in the controller as
 * `$this->request->id`.
 *
 * If you're using a relational database, such as MySQL, SQLite or Postgres, where the primary key
 * is an integer, uncomment the routes below to enable URLs like `/posts/edit/1138`,
 * `/posts/view/1138.json`, etc.
 */
Router::connect('/{:controller}/{:action}/{:id:\d+}.{:type}', array('id' => null));
Router::connect('/{:controller}/{:action}/{:id:\d+}');
```

Next, let's assume that there is already a view created to display the blog post at `views/blogposts/view.html.php`.  This means that when the controller's `view()` method is called, the aforementioned html view will be loaded.

In our controller, we can use the automatically generated `view()` method, provided you used the CLI to create the controller, which will get the id of the post from the URL.  If you didn't use the CLI to create the controller, then your view method should look something like this:

```
public function view() {
    $purchaseorder = PoHeader::find($this->request->id);
   	return compact('purchaseorder');
}
```

With these pieces in place, pointing the browser to `blogposts/view/1` will retrieve the blog post with id of 1 and render it into an HTML view, exactly as you expect.  If, however, you wish to render the same content in JSON as part of an API or other REST type application, then you can simply point the browser to `blogposts/view/1.json` and the data for the blog post will be rendered to the browser in a JSON format.   This is a very powerful feature as it allows you to develop a single controller method that can be quickly output into a variety of formats with very little effort.

## Other Ways to Change The Render Type

You can also statically define the type for a route by adding the 'type' key to the route definition:

```
Router::connect('/latest/feed', array(
	'Posts::index',
	'type' => 'xml'
));
```

If you'd rather use other information to convey the request type to Lithium (headers, GET variables, etc.) you can gather that information then set `$this->_render['type']` in the controller action.

Manual type rendering can also be done by handing the type's name to the render function:

```
$this->render(array('csv' => Post::find('all')));
```

## Handler Registration

As mentioned earlier, type handlers are registered by the `\net\http\Media` class. This is usually done in the `/config/bootstrap/media.php` bootstrap file. Be sure to uncomment the corresponding line in the main bootstrap file to enable this functionality.

Register your type by passing the relevant information to `Media::type()` inside the bootstrap. Here's what the general pattern looks like:

```
Media::type('typeName', 'contentType', array($options));
```

To give you an idea of how this process is completed, let's register a new handler for the BSON data type. If you're curious, BSON is a binary, serialized form of JSON used by MongoDB. Start by declaring a new media type in `/config/boostrap/media.php` and uncommenting the `media.php` line in your main bootstrap file:

```
Media::type('bson', 'application/bson', array());
```

This gets us pretty far. If you make a request with a .bson extension that matches a configured route (one with {:type} in it), Lithium will already hunt for a `.bson.php` template in the controller's template directory. You can continue to customize Lithium's behavior by utilizing the `$options` array you supply to `Media::type()`. After checking the API docs for `Media::type()`, we realize we can utilize a few options to make sure our response isn't housed in an HTML layout and use some functions we've defined for encoding and decoding the data:

```
Media::type('bson', 'application/bson', array(
	'layout' => false,
	'encode' => 'bson_encode',
	'decode' => 'bson_decode'
));
```

Try the request again, and you'll get your BSON-encoded data, minus the HTML layout. Note that the bson_* functions used in this particular example are part of the PECL MongoDB extension. Don't worry if you don't have it installed: the main point is to realize that you can tell `Media` exactly what function to use to render the data (including using closures).

For more ideas on configuring media types, see the documentation for `Media::type()`.