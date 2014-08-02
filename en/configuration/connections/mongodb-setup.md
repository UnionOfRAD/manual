# MongoDB Setup

MongoDB is a _document-oriented_ or _non-relational_ database. Document-oriented databases have many advantages over more traditional relational databases, including high performance and much simpler scalability. Many developers also like its schema-less nature, allowing the app to add fields to records as needed.

It is important to understand, however, that MongoDB is _not_ a replacement for relational databases. Relational databases maintain the advantage of much more powerful queries—especially when it comes to complex relations.

With that said, we'll leave it to the MongoDB folks to explain [how to set up MongoDB itself](http://www.mongodb.org/display/DOCS/Getting+Started).

Once you've got Mongo itself installed, follow the directions below, depending on your operating system:

## *NIX

For MongoDB + PHP goodness, you'll need the Mongo PECL module. For most *NIX systems, this is as easy as:

```
$ sudo pecl install mongo
```

Now add the following line to your php.ini:
```
	extension=mongo.so
```

That's it! Restart your web server, and you're done!

## Windows

The precompiled driver is available for VC8 and VC9.

Builds for each release are available on [ GitHub](http://github.com/mongodb/mongo-php-driver/downloads). They are all built for PHP 5.3, as PHP 5.2 is no longer supported with VC9.

Binaries for the latest code are also available:

    * VC8 Thread-Safe
    * VC8 Non-Thread-Safe
    * VC9 Thread-Safe
    * VC9 Non-Thread-Safe

Once you've identified the correct version for your system:

 * Download and extract the .zip file. Make sure PHP version_ matches the version of PHP you are running (e.g., if you are running PHP 5.3.2, download the driver for PHP 5.3).
 * Copy php_mongo.dll to your PHP extensions directory (see "extension_dir" in php.ini).
 * Add the following line to your php.ini:
```
extension=php_mongo.dll
```

Restart your web server, and you're done!

For more information about installing MongoDB alongside PHP, refer to the [PHP Manual](http://www.php.net/manual/en/mongo.installation.php).