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
	
	public function __construct(array $config = []) {
		$defaults = [
			'foo' => 'bar'
		];
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

$foo2 = new Foo([
	'foo' => '123'
]);
$foo2->baz();              // '123'

$foo3 = new Foo([
	'init' => 'false'
]);
get_class($foo3->service);  // PHP Warning...
get_class($foo->service);  // 'lithium\net\http\Service'
```

<div class="note note-version">
	The filtering system was overhauled in framework version 1.1. Filters are
	now managed via a dedicated Filters class, instead of before where Object managed 
	them.
</div>



