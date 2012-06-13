# Lithium & PHP 5.3

Lithium is the first PHP framework to take advantage of the new features available in PHP 5.3. Besides the performance benefits, there are a number of new concepts in 5.3+ that allow for some elegant development techniques.

There are a number of new features in PHP 5.3 that Lithium is taking full advantage of. This guide aims to introduce each  of these features and show examples of their implementation inside of Lithium architecture.

## Namespaces

[Namespaces](http://php.net/manual/en/language.namespaces.php) are used in Lithium to both isolate classnames and prevent name collisions, but they are also used for quick-and-easy class autoloading.

Here's an example of namespace usage in a Lithium controller:

{{{
namespace app;
use app\models\Post;
class PostsController extends \lithium\action\Controller {
    public function index() {
        $posts = Post:all();
        return compact('posts');
    }
}
}}}

Pretty straightforward. Using namespaces keeps your application's classes separate from the Lithium core and third-party plugins, allowing for commonly used class names (File, Folder, etc.) to be used in both places.

Lithium classes also follow the [PHP Standards Working Group namespace standard](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) in order allow for easy autoloading of class file, and cross-library interoperability. This keeps your class files clean from messy include statements.

## Anonymous Functions, Lambdas & Closures

Anonymous functions are a great tool for packaging up small bits of logic and passing them from place to place inside a system. PHP 5.3 now allows anonymous functions to be created and assigned to variables. For example:

{{{
$cube = function($value) {
    return ($value * $value * $value);
}
$result = array_map($cube, array(1, 2, 3));

// $result --> array(1, 8, 27)

$result = $cube(4);

// $result --> int 64
}}}

You can also use these anonymous functions in method parameters to create closures. Notice the use of use() in order to bring outside variables in scope for the closure logic:

{{{
$data = 'bad apple';
$result = array_filter(array(1, 2, 'bad apple'), function ($value) use ($data) {
    return ($value !== $data);
});

// $result --> array(1, 2)
}}}

Closures are used in a number of places in Lithium. One places is to create data validation rules on the fly:

{{{
Validator::add('validRole', function($value, $format, $options) {
    return(in_array($value, array('user', 'admin', 'editor')));
});
}}}

Closures are also used to create filters in Lithium: filters are a way to modify core functionality by inserting anonymous functions into an existing logic chain. More on that later.

## Late Static Binding

Also new in PHP 5.3 is the idea of late static binding. This feature allows us to create static classes that are smarter about class inheritance. It's little hard to explain in prose: here's some code that shows the difference between the new static:: and old self:: patterns:

{{{
<?php

class Person {
	public static function whoAmI() {
		return "Person";
	}
	public static function testSelf() {
		return self::whoAmI();
	}
	public static function testStatic() {
		return static::whoAmI();
	}
}

class Developer extends Person {
	public static function whoAmI() {
		return "Developer";
	}
}

echo Developer::testSelf();    // "Person"
echo Developer::testStatic();  // "Developer"

?>
}}}

Many classes (such as models) are statically accessed in Lithium. This allows for easy access, a stateless design, and better overall application design in general.

## Standard PHP Library (SPL)

PHP now comes with a standard library of objects that are meant to accomplish simple, oft-used tasks. Since you'll be developing with PHP 5.3, all of these classes will be at your fingertips. Iterators, exceptions, file handling, and data structure classes like stacks and queues are already built and ready to use.

## PHP OOP Syntax Enhancements

There are a few OOP "magic" enhancements we're also taking advantage of. The first is a new magic function named __callStatic(). Much like __call(), it is executed when an inaccessible method has been requested on an object, except it works for static objects.

The Lithium Validator class is a perfect example. If you recall, the Validator class statically provides data validation logic in your application. You can access this logic via rule(), or you can access the logic via the rule's name, which relies on __callStatic(). See these two equivalent examples:

{{{
Validator::rule('email', $email);
Validator::isEmail($email);
}}}

Another addition is the new __invoke() method. This method defines what happens when an object is called as if it was a function. This is what allows closures to work. Here's a basic example to show you how this works:

{{{
class WakeUpCall {
    function __invoke($message) {
        echo $message;
    }
}

$wakeUp = new WakeUpCall;
$wakeUp('Good Morning!');
}}}

A great __invoke() example in Lithium is how Controller has been architected. When a request is received, the Dispatcher passes it to a Controller object it has instantiated. The controller is then invoked and the proper action is called based on the routing information stored in the request object.

## PHAR 

Lithium also takes advantage of a new PHP feature that allows groups of files to be bundled together inside a single, includable archive. PHARs, or PHP archives allow developers to package libraries as a single file. 

PHARs are more that just compressed PHP files: they're accessible through a stream wrapper allowing you to include files inside the archive:

{{{
include 'phar:///path/to/archive.phar/file.php';
}}}

In fact, the phar:/// works very similar to the file:/// syntax you might already be used to. This allows you to use fopen() and opendir() on a phar much like you would on a conventional filesystem setup.

Lithium plugins are packaged and distributed as phars. This makes downloading a plugin from the central repository faster and more efficient, while at the same allowing Lithium to access every part of the plugin in it's compact, distributable state.

For more information, run `li3 help Library` at the console.

## New Extensions: Intl, Fileinfo, Sqlite3, Mysqlnd

There are a number of new extensions to PHP that are worth mentioning here as well. These new extensions can be used in Lithium applications, and are used by the core framework itself.

### Intl

The Internationalization extension contains a number of classes that enable developers to collate, sort, and format data according to a locale.

 - Collator: string comparison and sorting
 - NumberFormatter: As named, and also helps with parsing strings into numbers.
 - MessageFormatter: allows the embedding and extraction of localized data to and from messages
 - Normalizer: Unicode string normalizer (and verification)
 - Locale: interaction and lookup with local identifiers in various formats 
 - IntlDateFormatter: As named
 - ResourceBundle: access to binary resource file packages


### Fileinfo

This extension was created to make informed guesses about files based on their extension, and certain magic byte sequences at specific points in the file. This ability, while not a perfect indicator, still can give a developer a great idea about the data hidden inside of a file or stream.

{{{
<?php
$info = new \finfo(FILEINFO_MIME);
$result = $info->file(__FILE__);
var_dump($result);

// 'text/x-php; charset=us-ascii' 

$result = finfo_file(finfo_open(FILEINFO_MIME), __FILE__);
var_dump($result);

// 'text/x-php; charset=us-ascii'
?>
}}} 

### Sqlite3

SQLite is a self-contained, server-less, transactional database engine. You might already have experience with it if you've written desktop or mobile software. Since SQLite support ships with PHP 5.3, you can expect Lithium data source adapters for it as well.

### Mysqlnd

Before 5.3, all PHP-MySQL interaction was implemented using the services provided by the MySQL Client Library. Basically, if there was interaction with a MySQL server, the extension or feature was compiled against the MySQL Client Library.

This new extension is a native driver. Instead of using the Client Library, you can now use Mysqlnd to incorporate Mysql client and server communication. When building MySQL PHP extensions, you can now use this library. Features include:

 - Improved speed.
 - It's easier to compile (no longer an external dependency)
 - Improved persistent connections
 - Transparent client (same as mysql/mysqli)
 - mysql_fetch_all()
 - Compressed protocol support
 - Performance statistics (mysqi_get_X_stats())



