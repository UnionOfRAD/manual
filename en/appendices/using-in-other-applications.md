# Using li3 in external applications

Because li3 is so flexible, you can bootstrap the core framework into any application environment, and use just the features you need.  First, make sure you have installed li3 properly.

To use li3 classes, simply load the class that manages libraries (`lithium\core\Libraries`), and register the li3 library itself, as follows:

```php
include "/path/to/classes/libraries/lithium/core/Libraries.php";
lithium\core\Libraries::add('lithium');
```

<div class="note note-info">
	Note that `"path/to/classes"` may be relative to your include path.
</div>

## Integrating with CakePHP

CakePHP provides a convenient place to add this configuration: `app/config/bootstrap.php`.  Additionally, if you'd like to intermingle li3 classes (models and controllers) with their CakePHP counterparts, you can add the following (again, to `bootstrap.php`):

```php
define("LITHIUM_APP_PATH", dirname(__DIR__));
lithium\core\Libraries::add('app', array('bootstrap' => false));
```

This correctly locates your app directory in li3's class loader, and suppresses `app`'s default bootstrap file.  You may now import your namespaced app classes in your CakePHP classes, and use li3 and CakePHP models and other classes in parallel. Note that `LITHIUM_APP_PATH` should be defined _before_ `Libraries.php` is included.

## Defining data connections

If you plan on using li3's data layer, you need to create `connections.php` inside `LITHIUM_APP_PATH/config`.  If you are defining your connections dynamically, this file can be empty. By convention, however, this file is used to configure your database and web service connections.  See [`lithium\data\Connections`](http://li3.me/docs/lithium/data/Connections) for more information on how to do this.
