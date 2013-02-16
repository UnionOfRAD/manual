# Caching

The `Cache` static class provides a consistent interface to configure and utilize the different cache adapters included with Lithium, as well as your own adapters.

## Included Adapters
Lithium ships with several adapters for caching.  The adapters can be found in `lithium/storage/cache/adapter`.

* `Apc.php`: Alternative PHP Cache (APC)
* `File.php`: A minimal file-based cache (file)
* `Memcahce.php`: Memcache (libmemcached)
* `Memory.php`: A minimal in-memory cache
* `Redis.php`: Redis (phpredis)
* `Xcache.php`: XCache opcode cache

Each adapter provides a consistent interface for the basic cache operations of `write`, `read`, `delete` and `clear`, which can be used interchangeably between all adapters. Some adapters may provide additional methods that are not consistently available across other adapters.

## Enabling/Disabling Caching
To control whether or not caching is enabled, you can either comment or uncomment the following line in your application's `config\bootstrap.php` file:

{{{
/**
 * This file contains configurations for connecting to external caching resources, as well as
 * default caching rules for various systems within your application
 */
require __DIR__ . '/bootstrap/cache.php';
}}}

> NOTE:

> By default, caching is enabled in Lithium

## Setting Configuration Options

In most cases, you will configure various named cache configurations in your bootstrap process,
which will then be available to you in all other parts of your application. A simple example configuration:

{{{
Cache::config(array(
    'local' => array('adapter' => 'Apc'),
    'distributed' => array(
        'adapter' => 'Memcached',
        'host' => '127.0.0.1:11211',
    ),
    'default' => array('adapter' => 'File')
));
}}}

## More information
* [Cache Class API Documentation](http://lithify.me/docs/lithium/storage/Cache)
* [Cache Adapters API Documentation](http://lithify.me/docs/lithium/storage/cache/adapter)
