# Using Data Sources

## Introduction

Before we dive into the next section about models, it's important to understand how a model manipulates it's underlying data. In li3, this is done through a collection of classes called data sources.

While models expose a common API for your application to manipulate data, the purpose of a data source is to manage the direct connection details for a specific data storage medium. Apart from providing a common way to read and write, models use data sources to connect to, disconnect from, and describe their underlying services.

## Core Data Sources

li3 provides a number of data sources your models can utilize. The list is growing, but currently includes:

 * MySQL
 * SQLite3
 * CouchDB
 * MongoDB

## Data Sources and Models

Most of the time, using a data source is an automatic part of setting up a connection to your data store. This is usually done in the `config/bootstrap/connections.php` bootstrap file:

```php
use lithium\data\Connections;

Connections::add('default', array(
	'type' => 'MongoDb',
	'host' => 'localhost',
	'database' => 'my_lithium_app'
));

Connections::add('legacy', array(
	'type' => 'database',
	'adapter' => 'MySql',
	'host' => 'localhost',
	'login' => 'root',
	'password' => '53Cre7',
	'database' => 'my_older_lithium_app'
));
```

The `Connections` class handles adapter classes efficiently by only loading adapter classes and creating instances when they are requested (using `Connections::get()`).

Adapters are usually subclasses of `lithium\data\Source`. Once these connections have been made, the data source is available to your models. Since one of the connections defined above is named `default`, newly created models will automatically use that connection and its related data source adapter.

If you're using multiple data sources in your application, you can specify which to use in each model class, inside of the `$_meta` property. For example, if we had user profile data stored in the MySQL connection specified above as `legacy`, we'd create our model as follows:

```php
namespace app\models;

class UserProfiles extends \lithium\data\Model {

	public $_meta = array('connection' => 'legacy');
}
```
