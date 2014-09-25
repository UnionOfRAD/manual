# li3 on Litespeed

## Basic Setup

In order to provide li3 applications with clean URLs, li3 ships with a set of `.htaccess` files for use with Apache, which will handle URL rewriting.  Litespeed supports `.htaccess` files, so it should work similar to Apache.

Point Litespeed's Document Root to your application's `webroot` directory, which contains the `.htaccess` file with the rewrite rules.

## Production Setup

In production, it is recommended that you set `Allow Override` to `None` and instead place the rewrite rules in the `Rewrite` section of your Virtual Host in Litespeed's control panel.

In the `Rewrite` section, set `Enable Rewrite` to `Yes` and paste the following into the `Rewrite Rules` section:

```apache
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !favicon.ico$
RewriteRule . index.php [QSA,L]
```

Note: the `?url=$1` appended to the rewritten url in the `.htaccess` and Apache config examples is unnecessary.  Litespeed sets the appropriate environment vars so li3 can determine the requested path without needing to add a var to the query string.

## Path Issue and Fix

In order to determine the correct base url path for the app and for assets like scripts, css, and images, li3 first looks to the `$_SERVER['PHP_SELF']` server/environment variable.  The lsapi used by Litespeed to interact with PHP seems to set `$_SERVER['PHP_SELF']` to the `REQUEST_URI` rather than the actual PHP file that is handling the request.  This causes li3 to set the base path incorrectly.  To work around the issue, add the following to the top of your app's `config/bootstrap.php` file:

```php
/* fix rewrite issue with Litespeed **/
$_SERVER['PHP_SELF'] = str_replace('\\', '/', str_replace(
	$_SERVER['DOCUMENT_ROOT'], '', $_SERVER['SCRIPT_FILENAME']
));
$_ENV['PHP_SELF'] = $_SERVER['PHP_SELF'];
```

See the following docs for more info on how the base path and other environment values are determined by li3:

* [http://li3.me/docs/lithium/action/Request::_base()](http://li3.me/docs/lithium/action/Request::_base)
* [http://li3.me/docs/lithium/action/Request::env()](http://li3.me/docs/lithium/action/Request::env)
* [http://li3.me/docs/lithium/action/Request::_init()](http://li3.me/docs/lithium/action/Request::_init)


