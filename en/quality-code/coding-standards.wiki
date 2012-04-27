# Coding Standards

Lithium core developers use the following coding standards. We suggest you follow the same standards when developing Lithium applications and plugins.

There are some [wiki:tools tools] which help you keeping in line with the standards. We make use of the `Syntax` command which comes with the [QA plugin for Lithium](http://github.com/UnionOfRAD/li3_qa). Rules which have a corresponding check in Lithium QA are marked with the `?` symbol.

## Files

File names should be created with _CamelCase_ (according to the class they contain).

PHP files (files with the php suffix), should not have any whitespaces before or after the opening and closing php tag. The closing tag is preceded by an empty line.<sup>?</sup>

The closing php tag must always be on the last one of the file.<sup>?</sup> The closing php tag is **not terminated** with a newline character.<sup>?</sup> File **encoding is UTF-8**. The default permissions for folders are octal 0755, for files octal 0644.<sup>? partially</sup> Only if the file must be executable (i.e. from console) use octal 0744 for files.

### Lines

Lines must not exceed a limit of **100 characters**.<sup>?</sup> The soft limit is 80 characters. Lines should not have any trailing whitespace characters.<sup>?</sup>

### PHP tags

Always use long open tags:

	<?php

Never use short open tags:

	<?

Well, unless you are in a view and want the variable data to be automatically escaped. See the Lithium [template namespace documentation](http://lithify.me/docs/lithium/template) for more information on this exception.

	<?=$post->title;?>

Always include a closing tag:

	<?php

	namespace lithium\util;

	class Foo {
	   // ...
	}

	?>

## Indentation

**One tab** will be used for indentation. So, indentation should look like this:

	<?php
	// Base level
			// Level 1
				// Level 2
			// Level 1
	// Base level
	?>

Indentation is **always symmetrical**. Any opening syntax construct that begins on a new line should be matched with a closing construct (that also begins on a new line) indented at the same level.  Any lines enclosed by a beginning and ending construct should be indented exactly one tab in from the enclosing lines. If the contents of an enclosure are broken out over multiple lines, then no enclosed elements should be on the same line as the enclosing structures.

	// Bad.
	// (1) Array elements are indented too far.
	// (2) Closing parenthesis aren't on their own line.
	// (3) The first array element doesn't break to its own line.
	// (4) Either no space or too many spaces between key name and arrows.
	$params = array_merge($params, array('controller'=> $params['plugin'],
				'action'=> $params['controller'],
				'pass'   => array_merge($pass, $params['pass']),
				'named'=> array_merge($named, $params['named'])));

	// Good.
	$params = array_merge($params, array(
		'controller' => $params['plugin'],
		'action' => $params['controller'],
		'pass' => array_merge($pass, $params['pass']),
		'named' => array_merge($named, $params['named'])
	));

	// Bad (if any array elements have their own line, all should).
	$params = array('controller'=> $params['plugin'],
		'action'=> $params['controller'],
		'pass' => $params['pass']
	);

	// Good.
	$params = array(
		'controller' => $params['plugin'],
		'action' => $params['controller'],
		'pass' => $params['pass']
	);

	// Also okay, since the whole structure is defined on one line.
	$params = array('controller' => $params['plugin'], 'action' => $params['controller'], 'pass' => $params['pass']);


## Operators

All [operators ](http://php.net/manual/en/language.operators.php) must be surrounded by spaces<sup>?</sup>.

	$x = $y;
	'x' . 'y';
	$x || $y;
	$x ?: $y;
	$x ? $x : $y;
	$x += $y;
	$x - $y;
	$x - 23;
	function($x, $y = null) {};

However there are a few exceptions to the spacing rule<sup>?</sup>:

1. Increment and decrement operators must be directly followed by or following the variable.

2. The exclamation mark must be directly followed by the variable.

3. Colons appearing as part of a case condition must have no spaces surrounding them.

4. Labels must have no spaces surrounding them.

5. Negative **literal** integers or floats must have the minus sign directly attached.

6. Minus signs involved in negations of i.e. variables can be spaced or directly attached.

	$x--;
	$x++;
	!$x;
	-23;
	- $x;
	-$x;
	case 'x':
	:x

In order to increase readability of the code it is allowed to use spaces (not tabs) before operators in certain cases.

1. Multiple function calls.
	{{{
		$those    = examples($are);
		$the      = best($ones);
	}}}

2. Concatenating a long message.
	{{{
		$message  = 'this message is spread over ';
		$message .= 'multiple lines.';
	}}}


## Control Structures
Control structures are for example `if`, `for`, `foreach`, `while`, `switch` etc. Below, an example with `if`:

	if (($a == $b) || ($a == c)) {
		// Action 1.
	} elseif (!($c == $d) && ($a == $b)) {
		// Action 2.
	} else {
		// Default action.
	}

In the control structures there should be **1 (one) space before** the first parenthesis and **1 (one) space between** the last parenthesis and the opening bracket.

**Always use curly brackets** in control structures, even if they are not needed. They increase the readability of the code, and they give you fewer logical errors.<sup>?</sup>

Opening curly brackets should be placed on the same line as the control structure.<sup>?</sup>  Closing curly brackets should be placed on new lines, and they should have same indentation level as the control structure.<sup>?</sup> The statement included in curly brackets should begin on a new line, and code contained within it should gain a new level of indentation.

	// Bad - no brackets, badly placed statement.
	if ($foo == $bar) $foo++;

	// Bad - no brackets.
	if ($foo == $bar)
		$foo++;

	// Good.
	if ($foo == $bar) {
		$foo++;
	}

Instead of complex `elseif` constructs consider using stacked `if`s.

	if ($foo == $bar) {
		// Action 1.
	}
	if ($foo == 'bar') {
		// Action 2.
	}

### Switch Statements

The control portions of switch blocks should follow a consistent indent pattern, such that the closing `break` statement should be indented to the **same level** as the corresponding opening `case` statement.

	switch ($foo) {
		case "first":
			// Do something.
		break;
		case "second":
			// Do something.
		break;
		case "third":
		case "fourth":
			// Do another thing.
			return $bar;
	}

Return statements should always be indented one level greater than their corresponding `case` statements, even if the `return` is the last statement in the block.

### Ternary Operator

Ternary operators (`?:`) should only be used to set default values for variables or array keys which may be undefined or empty, or for simple true/false comparisons, and must fit on one line of code.  if a ternary exceeds these constraints, it should be broken out into a full `if` block.

Parentheses _may_ be added to the conditional expression or entire ternary statement for clarity.

## Variables

Variable names should be as descriptive as possible, but also as short as possible. Normal variables should start with a lowercase letter, and should be written in _camelBack_ in case of multiple words. Note that even variables containing objects must **not** start with a capital letter.

	$user = 'John';
	$users = array('John', 'Hans', 'Arne');

	$dispatcher = new Dispatcher();

Array keys used in `$options`/results arrays should be formatted according to the same rules as properties/variables.

## Constants

Constants should be defined in capital letters<sup>?</sup>. If a constant name consists of multiple words, they should be separated by an underscore character.

	define('FOO', 1);
	define('FOO_BAR_BAZ', 2);

## Casts

Casts must not have any whitespace inside the cast, must only use long type names and be **separated by one space** from the following variable or value.<sup>?</sup>

	// Bad.
	(bool) $result;
	( boolean ) $result;
	(boolean)$result;
	(boolean)  $result;

	// Good.
	(boolean) $result;
	(integer) $value;

## Functions

Function names should be written in _camelBack_.

	function longFunctionName() {
	  // ...
	}

Functions should be called **without space** between function's name and starting bracket. There should be **one space** between every parameter of a function call.

	$var = foo($bar, $bar2, $bar3);

Example of a function definition:

	function someFunction($arg1, $arg2 = '') {
		if ($foo == $bar) {
			$foo++;
		}
		return $bar;
	}

Parameters with a default value, should be placed last in function definition. Try to **make your functions return something**, at least `true` or `false` - so it can be determined whether the function call was successful.

	function connection(&$dsn, $persistent = false) {
		if (is_array($dns)) {
			$dnsInfo = &$dns;
		} else {
			$dnsInfo = BD::parseDNS(dns);
		}

		if (!($dnsInfo) || !($dnsInfo['phpType'])) {
			return $this->addError();
		}
		return true;
	}

Functions without body content should be written like so.

	function connection(&$dsn, $persistent = false) {}

## Namespaces

The namespace  declaration must appear directly after the opening php tag, separated by an empty line.<sup>?</sup> Any static dependencies **appear as one block** after the namespace declaration, separated by an empty line.

	<?php

	namespace lithium\util;

	use Exception;
	use lithium\util\Inflector;

### Singularity

Namespaces should be written in lowercase singular form, example:

	// Bad.
	namespace lithium\data\sources;

	// Good.
	namespace lithium\data\source;

There are a few exceptions to the singularity rule:

* ```tests\cases```,
* ```app\*``` (e.g., ```app\extensions```)
* ```<plugin>\*``` (e.g., ```myplugin\extensions```)

## Classes

### Name

Class names should be written in _CamelCase_, for example:

	class Foo {
	   // ...
	}

Class names should always be singular nouns, i.e. ```Dispatcher```, ```PagesController``` or ```Environment```, unless they are __collection__ classes.

### Member Visibility

All methods and properties must have  a visibility operator. The `var` operator is forbidden.<sup>?</sup>

A **protected** method or variable name start with a single underscore ("`_`").

	class Foo {
		protected $_iAmAProtectedVariable;

		protected function _iAmAProtectedMethod() {
			// ...
		}
	}

A private method or variables: **This is a framework, it should be extensible.**  As such, there's no reason to ever use private methods or variables.

### Anatomy

This defines the basic anatomy of a Lithium class, by the numbers:

	<?php

	namespace my\package; // (1)

	use my\package\util\Foo; // (2)
	use my\package\util\Bar;

	class AwesomeSocket extends \lithium\core\Object { // (3)

		protected $_classes = array( // (4)
			'query' => 'lithium\model\Query',
			'record' => 'lithium\model\Record'
		);

		public function __construct($config = array()) { // (5)
			// Minimum class bootstrap logic here...
			parent::__construct($config);
		}

		protected function _init() { // (6)
			// Any additional bootstrap logic here...
		}
	}

	?>

#### Legend

1. **Namespaces**: classes should always begin with a namespace definition.  This definition should always be preceded by a single newline, and as always, all PHP blocks should begin with a full `<?php` long tag.
1. **Static dependencies**: any basic, unchangeable utility classes which this class depends on or otherwise references should go here.  Class references should be fully resolved but must not have a leading backslash (`\`).
1. **Class definition**: this is the only place where classes may be used without being referenced or imported elsewhere (i.e. neither statically nor dynamically).  As with static dependencies, the parent class reference should be fully resolved.
1. **Dynamic dependencies**: any classes on which this class depends that can be changed through configuration.  These may be classes that this class instantiates, or that are used statically.  Keys of this array should be the name of the class dependency in camelBack format, and the value should be a fully-namespaced class reference.
1. **Constructor**: constructors must always accept exactly one parameter, which must be an array.  Only required class bootstrapping code should be put here.  This makes classes easier to test.  For static classes, this should be replaced with `public static function __init()`, which takes no parameters.  This method is called when the class is first loaded into memory.
1. **Initializer**: any additional class bootstrapping code goes here.  This method is automatically called by `Object::__construct()`, which may be disabled by passing `'init' => false` to `parent::__construct()`.

### Methods

A common method grammar and vocabulary should be adhered to as much as possible, and concise, memorable method names should be used in all other cases.  Prefixes and suffixes like `get*` and `set*` should always be avoided.  Method names should always communicate exactly what they do, without containing any redundancies, particularly with respect to the class name, i.e. `Dispatcher::dispatch()` (wrong).  Where possible, method names should also communicate whether the method is a "sub-routine" (performs an action) or a "pure function" (returns a value), for example:

 * `run()` (sub-routine)
 * `find()` (function)
 * `set()` (sub-routine)
 * `value()` (function)

If a function has a reciprocal sub-routine (or vice-versa), they should be combined where possible, using an optional parameter to differentiate.  The basic pattern is as follows:

	public function value($value = null) {
		if ($value === null) {
			return $this->value;
		}
		$this->value = $value;
	}

## Importing/Including

### Including

When including files with classes or libraries, use only and always the [require_once](http://php.net/require_once) function. Don't use brackets with the include statements as they are language constructs.<sup>?</sup>

### Importing

When importing classes, names should never be aliased unless absolutely necessary to avoid collision.

 * Good: `use lithium\util\Inflector;`
 * Bad: `use lithium\util\Inflector as Inflector;`

Except for the `extends` part of the class declaration, all namespaced class references should never be used inline in code.  Instead, the reference should be declared at the top of the class.

	// Good.
	class Bar extends \lithium\Foo {
		// ...
	}

	// Good.
	class Baz extends \lithium\core\Object {
		// ...
		$bar = new Bar();
		// ...
	}

	// Bad.
	class Baz extends lithium\core\Object {
		// ...
		$bar = new lithium\foo\Bar();
		// ...
	}

Names appearing after the `use` keyword are always fully qualified but must not have a leading backslash.<sup>?</sup>

 * Good: `use lithium\util\Inflector;`
 * Bad: `use \lithium\util\Inflector;`


### Errors and Exceptions

Exceptions must not be used for flow control (they're quite expensive as are errors, a stack trace must be generated).

Use exceptions only if the situation is really exceptional and doesn't occur under **normal** conditions (i.e. trying to read a file and file does not exist/is not readable).

Use [SPL exceptions](http://php.net/manual/en/spl.exceptions.php) or those provided by Lithium. 

#### Messages

Error and Exceptions messages roughly follow the code documenting standard. The following two sections are cited from that standard and especially apply.

All documentation must be written in English and must follow the basic rules of punctuation. **Sentences usually end with a period.** Sentences are separated by one space from each other. The first letter of a word must be capitalized if it's the beginning of a sentence, following a period or exclamation mark.

Markdown syntax can be used in all docblocks [here: messages, the author]. As a rule of thumb you should wrap in single backticks if you're talking about code. Please note that `null`, `false` and `true` must always be wrapped in single backticks.

As in other places variables which appear in strings where they get expanded must be wrapped in curly braces. Those variables are most often good candidates for also getting wrapped in backticks.

Sometimes a variable within a message may contain no content at all (in the event of an error condition that's possible). In order to still get meaningful messages places where dynamic content is inserted must be preceded by a term describing the type of content to be inserted. 

The message **must not** include the method/class name.

Examples of **good** exception messages:

	"Could not write template `{$template}` to cache."
	"Entity `{$entity}` not found."
	"Rule `{$rule}` is not a validation rule."
	'Could not route request.'

Examples of **bad** exception messages:

	"Could not write template {$template} to cache." // Literal not marked as such.
	"Could not write template '{$template}' to cache." // Using ticks instead of backticks.
	"`{$entity}` not found." // `$entity` may be empty.
