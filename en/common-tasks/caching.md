# Caching

The `Cache` static class provides a consistent interface to configure and utilize the different cache adapters included with li3, as well as your own adapters.

## Included Adapters
li3 ships with several adapters for caching.  The adapters can be found in `lithium/storage/cache/adapter`.

* `Apc`: Alternative PHP Cache
* `File`: A minimal file-based cache (file)
* `Memcahce` (libmemcached)
* `Memory`: A minimal in-memory cache
* `Redis` (phpredis)
* `Xcache`opcode cache

Each adapter provides a consistent interface for the basic cache operations of `write`, `read`, `delete` and `clear`, which can be used interchangeably between all adapters. Some adapters may provide additional methods that are not consistently available across other adapters.

## Enabling/Disabling Caching
To control whether or not caching is enabled, you can either comment or uncomment the following line in your application's `config/bootstrap.php` file:

```
/**
 * This file contains configurations for connecting to external caching resources, as well as
 * default caching rules for various systems within your application
 */
require __DIR__ . '/bootstrap/cache.php';
```

> NOTE:

> By default, caching is enabled in li3

## Setting Configuration Options

In most cases, you will configure various named cache configurations in your bootstrap process,
which will then be available to you in all other parts of your application. A simple example configuration:

```
Cache::config(array(
    'local' => array('adapter' => 'Apc'),
    'distributed' => array(
        'adapter' => 'Memcached',
        'host' => '127.0.0.1:11211',
    ),
    'default' => array('adapter' => 'File')
));
```

## More information

* [Cache Class API Documentation](http://li3.me/docs/lithium/storage/Cache)
* [Cache Adapters API Documentation](http://li3.me/docs/lithium/storage/cache/adapter)
