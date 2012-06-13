# Working with External Libraries

Lithium applications are made up of collections of _libraries_. A library is any discrete group of PHP class files; this includes any Lithium plugins, other class libraries like PEAR and Zend Framework, and other frameworks like Symfony 2. Even your application and the Lithium core itself are considered libraries in Lithium parlance.

## Basics

Libraries are managed by the `lithium\core\Libraries` class, which handles auto-loading, service location, and introspecting available classes. When loading classes, Lithium assumes a [PSR-0 compatible](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) class mapping structure. For more information, see the [documentation for the `Libraries` class](http://lithify.me/docs/lithium/core/Libraries).

The default Lithium distribution ships with two `libraries` directories: one located at the root of the distribution, and one located in the `app` folder (or any application you generate with the console tooling). Libraries can be installed into either of these directories, and both are used interchangeably throughout this guide. The global `libraries` directory is intended for libraries which are shared across multiple applications, whereas the local directory is intended for application-specific ones, and will override the global directory in the event of a conflict.

By convention, library configurations are added in `config/bootstrap/libraries.php`, and are loaded in your application's bootstrap process. However, there are no rules or requirements as to where or when this can happen. The default bootstrap files are simply that: a default; they're designed to be changed and re-organized as your application requirements dictate.

## Installation

Generally, there are two ways in which a library can be packaged and distributed. In the first, the library _is_ the distribution. In this case, the root of the PHP class libraries is the same as the root of the repository or distribution. An example of this is [the Lithium core](https://github.com/UnionOfRAD/lithium).

The alternative approach is for a distribution to contain its library in a subdirectory. Examples would be the [Zend Framework](https://github.com/zendframework/zf2/tree/master/library) or [Symfony 2](https://github.com/symfony/symfony/tree/master/src).

In the first case, installation is as simple as downloading (or cloning, or adding as a submodule) the library you wish to use into your `libraries` directory. When registering the library with [`Libraries::add()`](http://lithify.me/docs/lithium/core/Libraries::add(), no other configuration is necessary, since Lithium knows where to look for it, and how to map its classes.

Since that's not possible for libraries in the second case, we recommend an alternative installation: download (clone, etc.) the distribution to `libraries/_source`, and symlink the root of the library source into the `libraries` directory.

For example, the Zend Framework 1.11.7 distribution would be installed to `libraries/_source/ZendFramework-1.11.7`. Then, `libraries/Zend` would be symlinked to `libraries/_source/ZendFramework-1.11.7/library/Zend`, so that the root of the library source would map directly to where Lithium expects it to be.

This not only simplifies your configuration, but has the added benefit of allowing you to switch between different versions of vendor libraries for your application, just by changing a symlink. _Note_: While the `libraries` directories are used by convention, libraries can be referenced anywhere on the filesystem by including a `'path'` key in your `Libraries::add()` configuration.

## Configuration

The following examples should give you an overview of the various approaches to configuring class libraries in Lithium. The easiest type of library to configure are libraries written for PHP 5.3 or higher which conform to the above-referenced namespacing standard.

For example, the image manipulation library [ Imagine](https://github.com/avalanche123/Imagine), once installed according to the above (with `lib/Imagine` symlinked to `libraries/Imagine`), can be configured simply with the following:

{{{
Libraries::add('Imagine');
}}}

Again, because it conforms to the 5.3 namespacing standard, and because (using the symlink) Lithium knows how to locate its class files, no other configuration is required.

### PEAR

Since PEAR is typically installed into a system directory (i.e. `/usr/local/lib/php`), you should first symlink it to `libraries/PEAR`, then add the following:

{{{
Libraries::add("PEAR", array(
	"prefix" => false,
	"includePath" => true,
	"transform" => function($class, $config) {
		$file = $config['path'] . '/' . str_replace("_", "/", $class) . $config['suffix'];
		return file_exists($file) ? $file : null;
	}
));
}}}

There are a number of things to note about this configuration. First, while PEAR classes use package prefix, there is no standard vendor prefix, so the `'prefix'` setting must be overridden accordingly.

Second, PEAR classes depend on PHP's include path. Setting `'includePath'` to `true` adds the current library path to the include path. It can also be set to an absolute physical path if, for example, the library depends on the directory _above_ it being in the include path.

Finally, the `'transform'` key is used to manually map class names to file names. 

### Zend Framework 1.x

First, we recommend installing ZF1 from the unofficial Git mirror, at `git://github.com/Enrise/Zend.git`. This distribution contains only the class library, and as such, can be cloned (or submodule'd) directly into the `libraries` directory.

If you downloaded ZF1, or did an SVN checkout, you'll follow the alternative instructions above so that (for example, with ZF 1.11.7) you'll have `libraries/Zend` symlinked to `libraries/_source/ZendFramework-1.11.7/library/Zend`.

Once installed, ZF1 can be configured per the following. Again, because Lithium assumes 5.3 standard namespacing in all libraries, some special considerations are necessary for dealing with class libraries written for 5.2 and lower.

{{{
Libraries::add("Zend", array(
	"prefix" => "Zend_",
	"includePath" => LITHIUM_LIBRARY_PATH, // or LITHIUM_APP_PATH . '/libraries'
	"bootstrap" => "Loader/Autoloader.php",
	"loader" => array("Zend_Loader_Autoloader", "autoload"),
	"transform" => function($class) { return str_replace("_", "/", $class) . ".php"; }
));
}}}

First, because we're dealing with underscore-prefixed classes, we need to override the default prefix. Also, ZF1 depends on its parent directory being included in PHP's `include_path`, which we can tell `Libraries` to do, using the directory constants to provide an absolute path. Again, if your system is already configured for this, you can omit the `'includePath'` key.

Next, since ZF1 uses its own autoloader, we can tell `Libraries` to delegate to that one, first by setting its class file as the bootstrap file, then by indicating it should use the `Zend_Loader_Autoloader` class as the loader.

Most importantly, we're overriding how class names are transformed into path names, by passing in a custom function which transforms PEAR-style class names. Finally, to use classes in Zend's incubator, we can add a separate configuration for this.

Note, the following should appear **above** the primary ZF configuration, because they both have the same class prefix. However, this configuration will verify that a file exists before attempting to autoload it, allowing classes to "fall through" to other loaders.
{{{
Libraries::add("ZendIncubator", array(
	"prefix" => "Zend_",
	"includePath" => '/path/to/libraries/ZF_Install_Dir/incubator/library',
	"transform" => function($class) {
		$file = str_replace("_", "/", $class) . ".php";
		return file_exists($file) ? $file : null;
	}
));
}}}

### Zend Framework 2.x

Fortunately, installing and configuring ZF2 is quite a bit easier. It can be cloned (or added as a submodule) from the official repository at `git://github.com/zendframework/zf2.git` into `libraries/_source/zf2`. Then, symlink `libraries/_source/zf2/library/Zend` to `libraries/Zend`.

Because ZF2 uses the same class naming scheme as Lithium, configuring it is quite a bit easier:

{{{
Libraries::add("Zend");
}}}

As with ZF1 above, using Zend's native autoloader is technically optional. Here, it is omitted specifically because both Lithium and ZF2 class loading rules were written to the same specification, and using this approach, Lithium can cache ZF class paths along with its own.

### Usage example: Zend Framework 1.x / 2.x

Once you've properly installed and configured your chosen version of Zend Framework, you can use its classes as you would any other:

{{{
namespace app\controllers;

/**
 * Import the class names. Use this style for static dependencies.
 */
use Zend_Mail_Storage_Pop3; // For ZF1
use Zend\Mail\Storage\Pop3; // For ZF2

class EmailController extends \lithium\action\Controller {

	/**
	 * Use this style for dynamic dependencies.
	 */
	protected function _init() {
		$this->_classes += array(
			'pop3' => 'Zend_Mail_Storage_Pop3' // ZF1
			'pop3' => 'Zend\Mail\Storage\Pop3' // ZF2
		);
	}

	public function index() {
		// If used statically:
		$mail = new Zend_Mail_Storage_Pop3(array(
			'host' => 'localhost', 'user' => 'test', 'password' => 'test'
		));

		// If used dynamically:
		$mail = $this->_instance('pop3', array(
			'host' => 'localhost', 'user' => 'test', 'password' => 'test'
		));

		return compact('mail');
	}
	
}
}}}


### TCPDF

Some legacy vendor libraries have no consistent class-to-file mapping scheme whatsoever. These libraries must be mapped by hand. Below is an example of a small library (TCPDF), containing just a few files.

 _Note_: For larger libraries, generating the map by hand can be tedious. Check out [the `Inspector` class](http://lithify.me/docs/lithium/analysis/Inspector), which can be used to introspect classes and files to generate a map automatically.

{{{
Libraries::add('tcpdf', array(
	'prefix' => false,
	'transform' => function($class, $config) {
		$map = array(
			'TCPDF2DBarcode'     => '2dbarcodes',
			'TCPDFBarcode'       => 'barcodes',
			'TCPDF_UNICODE_DATA' => 'unicode_data',
			'PDF417'             => 'pdf417',
			'TCPDF'              => 'tcpdf',
			'QRcode'             => 'qrcode'
		);
		if (!isset($map[$class])) {
			return false;
		}
		return "{$config['path']}/{$map[$class]}{$config['suffix']}";
	}
));
}}}

Again, the lack of vendor prefix is denoted by setting `'prefix'` to `false`. Next, the class-to-file mapping function is passed in the `'transform'` key, which contains an array mapping class names to file names. These are then used to generate and return a full physical path to the correct file.