# Lithium Structure

Before you get started with Lithium, it's important to know how your application's main folder is structured, and where everything should go. Clone yourself a copy of the main repo, and you'll find an app folder structure that contains:

* config
* controllers
* extensions
* libraries
* models
* resources
* tests
* views
* webroot

Let's tackle each of these folders one-by-one, covering what each is used for, and how you can best organize your application.

## Config

The `config` folder contains three main pieces: bootstrap files, your connections information, and your routes definitions.


### Bootstrapping Lithium
The main bootstrap file in the config folder is `bootstrap.php`. This file is second file PHP will execute in any request cycle, and it loads up the Lithium framework along with any other extras that you define there. As such, it's a great place to place initial application configuration.

The best practice for bootstrapping parts of your application is by writing specific bootstrap files and placing them in `config/bootstrap/filename.php`. Once written, include the configuration into the main bootstrap file:

{{{
// Adding my custom configuration or initialization...
require __DIR__ . '/bootstrap/myconfig.php';
}}}

### Connections

The connections.php file in config lists the connections you have to external data resources, most often some type of database.

Connections should be listed in this file, as calls to `Connections::add()`, like so:

{{{
Connections::add('default', array(
    'type' => 'database',
    'adapter' => 'MySql',
    'host' => 'localhost',
    'login' => 'root',
    'password' => '',
    'database' => 'my_blog'
));

Connections::add('couch', array(
    'type' => 'http', 'adapter' => 'CouchDb', 'host' => '127.0.0.1', 'port' => 5984
));

Connections::add('mongo', array('type' => 'MongoDb', 'database' => 'my_app'));
}}}

The particulars on the adapter will shape how the connection definition is put together, but this list should constitute what you've got in `connections.php`.

### Routes

Routes definitions is how you inform the framework how URLs and bits of code match up. At the most basic level, it's how you tell Lithium which controller should respond to a request to a given URL.

For example, I could specify that any request  to `/login` is handled by `UsersController::login()` like this:

{{{
Router::connect('/login', array('controller' => 'users', 'action' => 'login'));
}}}

More on routes later, but this file should house all such configuration information.

## Controllers

Application controllers are housed here. Each controller is named in CamelCase, and placed here.

## Extensions

The extensions folder is meant to store extension classes that you've created for your application. Custom helpers, adapters, console commands and data classes.

### Adapter

A number of Lithium classes extend the `lithium\core\Adaptable` static class. This class allows you to take a number of different approaches and easily switch out implementations for an application task. Authentication, session handling, and caching are a few examples of Lithium functionality that have been made adaptable.

Custom application adapters should be placed in this folder, organized by subfolder. For example, if I created a custom Auth adapter based on my LDAP setup, I'd create the implementation in `/extensions/adapter/auth/Ldap.php`.

### Command

Custom console applications are placed in `command`.  Commands you create should be built similar to those you see in `/lithium/console/command`.

As an example, say I created a custom mailer queue, and I need to create a command to fill up the queue with announcement emails and trigger the send process. I'd create a new class called `QueueFiller` that extends `lithium\console\Command`, and place it in `app/extensions/command/QueueFiller.php`.

Once there, running `li3 QueueFiller` from the command line will trigger the logic I've written.

### Data

Occasionally a new application will require custom data classes to work with a storage engine not already accommodated for by core classes.

If you've got some proprietary database, and it's result objects or query structures are very customized, you'll extend the classes in ` lithium/data` and place them here for use in your application.

### Helper

Custom view helpers are common need. These classes should extend `lithium\template\Helper` and be placed in this folder. 

## Libraries

The `libraries` folder is meant to house entire Lithium applications—or plugins—that your main application makes use of. 

Any plugins you download from the [ Laboratory](http://lab.li3.me/), or pull down via the Library li3 command should end up here.

Third-party, non-Lithium libraries should be placed here as well. If you're integrating with Facebook, or using part of the Zend Framework in your application, place those files here as well.

## Models

Application models are stored here, in CamelCase naming format.

## Resources

The resources folder is used by a Lithium application as a non-web viewable storage place for application data. By default globalization files (such as .po string files) and temporary application files are stored there.

As you build your application, the `resources` folder is a great place for other application data like custom cache files, SQLite databases, or a temporary spot for uploaded files.

The `resources` is a place where the web server has write access, so make sure to keep that in mind when securing your application.

## Tests

The `tests` folder is where the unit testing framework files for your application reside. The testing setup for Lithium is covered elsewhere, but let's cover what ends up in each of the testing subfolders here.

### Cases

All of the actual unit testing logic for the main MVC classes in the application go in the `cases` folder. Subfolders should already exist inside of `cases` to prompt you where each sort of test case should go.

### Integration

Unit tests in the `integration` folder test the interaction between two or more classes, or abstract class implementations. 

### Mocks

Classes in `mocks` are used to facilitate certain unit tests. For example, if I'm testing a custom model I've created, I can create a new class in `mocks/data/MockSource.php` and use it in my model unit testing logic. 

It is recommend that you use subfolder organization that mimics your application's and Lithium's namespacing and folder structure.

## Views

The `views` folder contains any pieces of view-level code in your application. The two main core folders here to note are `elements` and `layouts`. Other view folders here contain views organized by controller name. 

### Elements

Elements are small bits of view code that are used in multiple views. A common navigational element, menu, or form might end up as an element in this folder. Element files are suffixed with a compound extension: the first showing the type of view code, and the last as `.php`. 

Example element filenames might include `nav.html.php`, `animateLink.js.php`, or `header.xml.php`.

### Layouts

Views (and the elements they may contain) are rendered inside of view layouts. The layouts for your application reside in this folder, named similar to elements.

A layout named `default` will be used to render a view if none other has been specified by the controller.

### Controller View Folders

Along with the `elements` and `layouts` folders, many other controller-specific view folders will reside here.

One default example is the `pages` folder here that holds all the views for the core `PagesController`. As you create new controllers and views for your application, create new folders here, named after your controllers.

## Webroot

The `webroot` folder of your application is (ideally) the only place in your application that is web-visible. The `index.php` file takes care of bootstrapping Lithium, and the rest of the folders and files here are used for static content delivery. 

Things like JavaScript files, images, Flash objects, and movies should end up here for easy access for your web server to deliver to the client.