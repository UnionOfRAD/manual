# Constructors and Initialization

The base class in li3's hierarchy, `lithium\core\Object`, forms the basic logic for all of the concrete classes in the framework. This class defines several conventions for how classes in li3 should be structured:

 * Universal constructor
 * Object configuration
 * Object initialization
 * Filtering
 * Testing utilities

## Universal Constructor

The `Object` class features a unified constructor, which all classes share. The function takes a single argument, `$config` which is an array of properties that is automatically saved to the `$_config` property of the current object. This approach allows for short constructor signatures and creates a unified way to override object configuration, or set object defaults when subclassing.

## Object Configuration & Initialization

Constructor logic should be kept to a minimum. One of the last steps in the unified constructor is to call the objects `init()` method. Here's where you do your heavy lifting. 

If you need to manipulate an object in its un-initialized state, pass `false` as the value to the `init` key to the constructor. This prevents `init()` from running, and is especially handy when building test cases.

```php
namespace app/extensions;

use lithium\net\http\Service;

class Foo extends \lithium\core\Object {
	public $service;
	
	public function __construct(array $config = array()) {
		$defaults = array(
			'foo' => 'bar'
		);
		parent::__construct($config + $defaults);
	}
	
	public function _init() {
		parent::_init();
		$this->service = new Service();
	}
	
	public function baz() {
		echo $this->_config['foo'];
	}
} 
```

```php
use app\extensions\Foo;

$foo = new Foo();
$foo->baz();               // 'bar'

$foo2 = new Foo(array(
	'foo' => '123'
));
$foo2->baz();              // '123'

$foo3 = new Foo(array(
	'init' => 'false'
));
get_class($foo3->service);  // PHP Warning...
get_class($foo->service);  // 'lithium\net\http\Service'
```

## Filtering

The `Object` class also allows subclasses to filter their methods. An entire guide has been dedicated to filters, and you'll find examples and explanations later in this chapter.

Long story short, a filterable method is created by encapsulating the main logic for a method inside a closure. That closure is then passed to `Object::_filter()` (or `StaticObject::_filter()` for static objects) to allow for chain management. 

Subclasses of `Object` use the filter methods to allow other developers to wrap logic around their own without cluttering the object's API.



