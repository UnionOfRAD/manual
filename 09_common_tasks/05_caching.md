# Caching

The `Cache` static class provides a consistent interface to configure and utilize the different cache adapters included with li3, as well as your own adapters.

## Included Adapters

The framework ships with several adapters for caching. These adapters can be found in
`lithium/storage/cache/adapter`. Each adapter has its own special characteristics, 
pick one (or two) dependent on your specific use case.

* `Memcache` - A libmemcached based adapter, highly recommended.
* `Apc` - Can be used if you're on an older PHP version and cannot use memcached, but have APC or APCu available.
* `Redis` - Recommended if you're already using redis for other tasks.
* `File` -  A minimal file-based cache, good if your app lives in constrained environment or you'd like to cache BLOBs.
* `Memory` - A minimal in-memory cache, good for testing purposes.
* `Xcache` - An alternative to `Apc`, not recommended as support might be phased out.

Also see the [Cache Adapters API Documentation](http://li3.me/docs/api/lithium/latest:1.x/lithium/storage/cache/adapter) for more information.

## Enabling/Disabling Caching

To control whether or not caching is enabled, you can either comment or uncomment the
following line in your application's `config/bootstrap.php` file:

```php
/**
 * This file contains configurations for connecting to external caching resources, as well as
 * default caching rules for various systems within your application
 */
require __DIR__ . '/bootstrap/cache.php';
```

<div class="note note-info">
	By default, caching is enabled in the framework.
</div>

## Configuration

In most cases, you will configure various named cache configurations in your bootstrap
process, which will then be available to you in all other parts of your application. A
simple example configuration:

```php
Cache::config([
    'local' => [
        'adapter' => 'Apc'
    ],
    'distributed' => [
        'adapter' => 'Memcached',
        'host' => '127.0.0.1:11211'
    ],
    'default' => [
        'adapter' => 'File',
        'strategies => ['Serializer']
    ]
];
```

### Strategies

Each cache configuration can be configured with _strategies_. These influence how values are read and written
into the cache. Some adapters already handle serialization for you, others like `File` do not do this. This
is why we configure the `File` adapter using the general `Serializer` strategy. Other stratgies can be found
in the [Cache Strategies API Documentation](http://li3.me/docs/api/lithium/latest:1.x/lithium/storage/cache/strategy).

### Scoping

Adapter configurations can be scoped, adapters will then handle the
namespacing of the keys transparently for you. This prevents caches
from "stepping on each others toes".

```
Cache::config([
    'primary'   => ['adapter' => 'Apc', 'scope' => 'primary'],
    'secondary' => ['adapter' => 'Apc', 'scope' => 'secondary']
];
```

## General Operation

Adapters provide a consistent interface for basic cache operations (`write`, `read`,
`increment`, `decrement`, `delete` and `clear`), which can be used interchangeably between
all adapters. Some adapters may provide additional methods that are not consistently
available across other adapters.

### Writing to the Cache

All cache operations take the name of the configuration as their first argument. This
allows you to use the best cache configuration for your use cache.

```php
// Will store the value `'bar'` under the `foo` key, using the default expiry.
Cache::write('default', 'foo', 'bar');
```

<div class="note">
	The read/write and delete methods can handle multi keys/values or so called batch operations. 
	Simply pass an array of keys (and value pairs) to the respective method.
</div>

To specify an **expiry**, use the 4th parameter of the method. Expiry time is a `strtotime()`
compatible string. Alternatively an integer denoting the seconds until the item expires
(TTL). If no expiry time is set, then the default cache expiration time set with the cache
adapter configuration will be used. To persist an item use `Cache::PERSIST`.

```php
Cache::write('default', 'foo', 'bar', '+1 hour');
Cache::write('default', 'foo', 'bar', Cache::PERSIST);
```

### (Atomically) Increment/Decrement Values

Two specialized methods for writing to the cache are `Cache::increment()` and `Cache::decrement()`. These
can be used i.e. if you want to increase a counter. Some adapters handle these operations atomically others
can't. Please check your adapter configuration for details.

```php
Cache::increment('default', 'pageviews'); // increment count by one
Cache::increment('default', 'pageviews', 2); // increment count by two
```

### Reading from the Cache

Reading from cache is pretty is and after reading the above you should already be able to
guess, how this works.

```php
// Will read the value under the `foo` key, if isn't found returns `null`.
Cache::read('default', 'foo');
```

## Caching BLOBs

<div class="note note-version">This feature is available beginning with 1.1.0.</div>

BLOBs are binary large objects or simply put: _files_. The file cache adapter is capable
of storing BLOBs using the following configuration. 

```php
Cache::config([
	'blob' => [
		'adapter' => 'File', 
		'streams' => true
	]
]);
```
Imagine - upon user request - a PDF is compiled. This requires quite a 
bit of CPU time and memory. Upon following requests for the same PDF you
want to save some cycles and return the file from cache. This example
shows you how.

```php
// We will need a stream handle we can write to and read from.
$stream = fopen('php://temp', 'wb');

// Pseudocode; generate a PDF then store it in the stream. 
$pdf->generate()->store($stream);

// We must rewind the stream, as Cache will not do this for us.
rewind($stream);

// Store the contents of $stream into a cache item.
Cache::write('blob', 'productCatalogPdf', $stream);
```

```php
// ... later somewhere else in the galaxy ...
$stream = Cache::read('blob', 'productCatalogPdf');

// Output $stream to the client.
// echo stream_get_contents($stream);
```


