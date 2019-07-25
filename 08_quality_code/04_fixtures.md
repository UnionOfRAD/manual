# Fixtures Management for the li₃ framework.

This plugin provide fixtures managment. Should work with any kind of `Source`
adapters. The fixture class support the following datasource's hints:

- If `Source::enabled('schema')` returns `true`, the `Fixture` manage schema
  (i.e create/drop) via `Source::createSchema()` & `Source::dropSchema()`.

- If `Source::enabled('sources')` returns `true`, the `Fixture` allow soft drop
  (i.e safe options).

## Installation

The preferred installation method is via composer. You can add
the library as a dependency via:

```
composer require unionofrad/li3_fixtures
```

li₃ plugins must be registered within your application bootstrap phase 
as they use a different (faster) autoloader. 

```php
Libraries::add('li3_fixtures')
```

The official manual has more information on 
[how to register the plugin](http://li3.me/docs/manual/installation/plugins.md) 
with the app or use alternative installation methods (i.e. via GIT).

## API

### The Fixture class

#### Methods:

- Fixture::create($safe); //Create the source only
- Fixture::save($safe); //Create the source + save the fixture's records in.
- Fixture::drop($safe); //Drop the source.
- Fixture::populate($records); //Insert a record in the database
- Fixture::alter($mode, $fieldname, $value); //Altering the schema before `::create()/::save()`.

Simple example of unit test:

```php
<?php
//app/tests/cases/models/SampleTest.php
namespace app\tests\cases\models;

use li3_fixtures\test\Fixture;

class SampleTest extends \lithium\test\Unit {

	public function testFixture() {
		$fixture = new Fixture([
			'connection' => 'lithium_mysql_test',
			'source' => 'contacts',
			'fields' => array(
				'id' => array('type' => 'id'),
				'name' => array('type' => 'string')
			),
			'records' => array(
				array('id' => 1, 'name' => 'Nate'),
				array('id' => 2, 'name' => 'Gwoo')
			)
		]);

		$fixture->save();

		$fixture->populate(['id' => 3, 'name' => 'Mehlah']);

		$fixture->drop();
	}
}
?>
```

### The Fixtures class

`Fixture` is a kind of `Schema` which contain records and a source name or a reference to a model.
So let save the above fixture in a class.

```php
<?php
//app/tests/fixture/ContactsFixture.php
namespace app\tests\fixture;

class ContactsFixture extends \li3_fixtures\test\Fixture {

	protected $_model = 'app\models\Contacts';

	protected $_fields = array(
		'id' => ['type' => 'id'],
		'name' => ['type' => 'string']
	);

	protected $_records = [
		array['id' => 1, 'name' => 'Nate'],
		array['id' => 2, 'name' => 'Gwoo']
	];
}
?>
```

```php
<?php
//app/models/Contact.php
namespace app\models;

class Contacts extends \lithium\data\Model {}
?>
```

If you have numbers of fixtures, it will be interesting to use the `Fixtures` class.

Example of use case:

```php
<?php
//app/tests/integration/Sample2Test.php
namespace app\tests\integration;

use li3_fixtures\test\Fixtures;
use app\models\Contacts;
use app\models\Images;
// and so on...

class Sample2Test extends \lithium\test\Unit {

	public function setUp() {
		Fixtures::config([
			'db' => [
				'adapter' => 'Connection',
				'connection' => 'lithium_mysql_test',
				'fixtures' => array(
					'contacts' => 'app\tests\fixture\ContactsFixture',
					'images' => 'app\tests\fixture\ImagesFixture'
					// and so on...
				)
			]
		]);
		Fixtures::save('db');
	}

	public function tearDown() {
		Fixtures::clear('db');
	}

	public function testFixture() {
		var_export(Contacts::find('all')->data());
		var_export(Images::find('all')->data());
	}
}
?>
```

Ok so why it's better to set the `Fixture::_model` instead of `Fixture::_source` ? Long story short,
models had their own meta `'connection'` value. If a fixture is "linked" with a model, it will
automagically configure its meta `'connection'` to the fixture's connection when is created or saved.

Example:

```php
<?php
Fixtures::save('db', array('contacts'));
//The line bellow is not needed since Contacts have been configured by ContactsFixture.
Contacts::config(['meta' => ['connection' => 'lithium_mysql_test']]);
var_export(Contacts::find('all')->data());
?>
```

### Advanced use case

For interoperability, sometimes it's usefull to adjust fixtures according a datasources.

You can alter `Fixture`'s instance before creating it like the following use case:

```php
<?php
$fixture->alter('add', [
	'name' => 'enabled',
	'type' => 'boolean'
]); //Add a field

$fixture->alter('change', [
	'name' => 'published',
	'value' => function ($val) {
		return new MongoDate(strtotime($val));
	}
]); //Simple cast for fixture's values according the closure

$fixture->alter('change', [
	'name' => 'id',
	'to' => '_id',
	'value' => function ($val) {
		return new MongoId('4c3628558ead0e594' . (string) ($val + 1000000));
	}
]); //Renaming the field 'id' to '_id' + cast fixture's values according the closure

$fixture->alter('change', [
	'name' => 'bigintger',
	'type' => 'integer',
	'use' => 'bigint' //use db specific type
]); //Modifing a field type

$fixture->alter('drop', 'bigintger'); //Simply dropping a field
?>
```

Note :

You can recover a specific fixture's instance from `Fixtures` using:

```php
<?php
$fixture = Fixtures::get('db', 'contacts');
?>
```

