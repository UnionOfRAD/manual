# FAQ

### PHP short tags in templates? Haven't those been deprecated for, like, ever?

You may have noticed that Lithium templates use PHP short tags (i.e. `<?=`). Don't worry though, these aren't actually interpreted by PHP. Instead, Lithium uses an internal template compiler to change these tags into full `<?php echo ... ?>` statements, with the added benefit of automatic content escaping, so you don't have to think about XSS attacks. Also, the compiler is smart enough to recognize content coming from helpers, etc., so it won't double-escape. For more info, check out the documentation on [the templating system](http://li3.me/docs/lithium/template).

### You guys use statics all over the place. Aren't those really hard to test?

Strictly speaking, statics are quite easy to test. Lithium borrows several concepts from _functional programming_ in order to ensure highly testable code throughout the entire framework, as well as applications built on top of it. In order to understand some of these concepts, first we have to define a few terms:

 _State_: An essential part of writing software, [Wikipedia defines state](http://en.wikipedia.org/wiki/Program_state) as "a snapshot of the measure of various conditions in the system".  When developing PHP applications, this varies by context / scope: when the web server first loads up `index.php`, the state is defined by the data used to request the script, including GET or POST data, and any system information exposed in `$_SERVER` or `$_ENV`, as well as any other super-global variable. State can also include non-obvious things like the current date and time.  Within a method, the state consists of any parameters passed in, anything within the object to which the method is bound (i.e. `$this`), and any other data defined or made available in the global scope.

 _Side-effect_: A concept closely related to state, _side effects_ are changes that a method or other routine makes to things outside its own scope. Side effects include things like changing the values of global variables, object properties (i.e. `$this`), modifying data in a database or filesystem, changing the value of a parameter passed by reference, or echoing output.

 _Mutability_: Mutability is the quality of something which is changeable. The basic incarnation of mutability in PHP is a variable or object property. By contrast, something which is _immutable_ is not changeable, i.e. a global or class-level constant. Therefore, _mutable state_ is an element of application state (see above) that can be changed. These changes are what produce side-effects. This may seem obvious, but becomes more important later on.

 _Referential transparency_: A quality of a method or function which prescribes that its return value can only be dependent on its parameters, and that it has no effect on the state of the program outside its own scope. Methods which are referentially transparent are very easy to test, since they do not require the setup of any outside state; a given set of parameters should always have the same outcome. Referentially transparent functions are also easy to cache, since multiple calls (with the same parameters) always return the same value. This process is known as _memoization_.

Getting back to our discussion, the problems with testing statics that people typically refer to actually have to do with _mutable global state_, i.e. a static method that depends on some external (usually global) piece of information which is subject to change by outside forces. The epitome of this anti-pattern is the Singleton. The idea behind singletons is that you have one (and only one) globally-available instance of an object at any one time.

The essential problem with this is that there's often no way to determine at any given time what the value of a singleton's attributes could be, because any part of your application has access to modify them. **This exemplifies the root cause of almost all software logic bugs.** The only solution is to design the singleton such that, once created, its attributes cannot be changed (i.e., make it _immutable_). Alternatively, re-design the affected architecture so that a singleton is not required.

Fortunately, all static classes in Lithium are either composed of _referentially transparent_ methods, or methods whose usage patterns are oriented around _immutability_. Some examples of referentially transparent methods are `lithium\util\String::insert()`, which inserts values into a template string, or `lithium\util\Inflector::camelize()`, which produces a version of a word or phrase. These functions have no external side-effects, and produce predictable output based on their parameters.

Examples of immutable static classes would be any class extends [`Adaptable`](http://li3.me/docs/lithium/core/Adaptable), i.e. `Cache`, `Connections`, etc. These classes model and provide access to system-level resources such as database connections, user sessions, and caching configurations. These classes are configured with information on how to access and operate on their respective resources using the `config()` method of each class. This happens once and only once, during the application's bootstrap process. Subsequent access to those classes (i.e. through the `adapter()` method) always returns the same adapter instance with the same attributes. This avoids the problem of having our application's state change out from under us.

The other issue often referred to when testing statics is that they're difficult to mock, or replace dependencies. Consider the following:

```php
class A {
    public static function foo() {
        // return some calculated value
    }
}

class B {
    public static function bar() {
        $result = A::foo();
        // perform some calculation on $result
        return $result;
    }
}
```

In this example, every call to `B::bar()` results in a call to `A::foo()`. It is impossible to test `B` in isolation, because it is impossible _not_ to also call `A`. In PHP 5.2 and below, there was no solution to this. However, PHP 5.3 allows "dynamic" static method calls, which enable the following:

```php
class B {
    public static function bar($dependency) {
        $result = $dependency::foo();
        // perform some calculation on $result
        return $result;
    }
}
```

This allows `A` to be swapped out for another class, and makes `B` much easier to test, since `$dependency` can be the name of a mock class that can act as a control during testing. This also has the pleasant side-effect of making `B`'s design more flexible.

Lithium addresses this issue of dependencies in a uniform way, using the protected `$_classes` attribute. Consider the following snippet from the `lithium\action\Dispatcher` class:

```php
class Dispatcher extends \lithium\core\StaticObject {

	// ...
	protected static $_classes = array(
		'router' => 'lithium\net\http\Router'
	);
	// ...
}
```

The `Dispatcher`'s dependencies may then be dynamically configured to use a different routing class with the `config()` method. Internally, calling the `Router` looks like this:

```
$router = static::$_classes['router'];
$result = $router::process($request);
```

Not all dependencies are configured this way, but it is the predominant convention throughout the framework. In almost all other cases, dependencies which cannot be changed are dependencies on utility methods, which are almost always referentially transparent. Because of this, these hard-coded dependencies add no difficulty or complexity in testing.

### I just installed Lithium, and I'm getting a fatal error that looks like this...

**Function name must be a string in .../lithium/util/collection/Filters.php on line ...**

This is happening because you have [ eAccelerator](http://eaccelerator.net/) installed. eAccelerator is an optimizer / opcode cache for PHP which does not fully support PHP 5.3. The solution is to disable it, and switch to a working, better-supported accelerator, like APC or Xcache.

If you didn't know you had eAccelerator installed, it's because you're running MAMP, which comes pre-bundled with eAccelerator. In this case, the solution is to man up (or woman up) and [get Homebrew](https://github.com/mxcl/homebrew). You'll be glad you did.

### Why are there 'libraries' folder in both the root directory and the app directory?

Some libraries are used by a single app or on an app-by-app basis. These apps store libraries in the `/app/libraries` directory. Other libraries are shared by one or more apps. The `/libraries` directory is provided to make administration of these libraries easier. There is also a (typically negligible) disk space savings as any given library isn't duplicated in the `/app/libraries` directories for multiple applications.

