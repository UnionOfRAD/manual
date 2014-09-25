# Troubleshooting li3 Installations

This quick list is meant to cover common problems in installing li3.

## Internal Server Error

Internal server errors are usually a result of bad .htaccess configurations. Make sure unmodified copies of the .htaccess files from the lithium repo are in these places:

 * `/.htaccess`
 * `/app/.htaccess`
 * `/app/webroot/.htaccess`

You might also be running in a directory on your web server that is already under rewrite rules (often URLs that include your username such as http://username.example.com or http://example.com/~username/). This may cause 500 Internal server errors, or in some cases, cause a redirect loop.

In this case you'll need to adjust your .htaccess files to include a RewriteBase directive:

 * `/.htaccess` => `RewriteBase /`
 * `/app/.htaccess` => `RewriteBase /app/`
 * `/app/webroot/.htaccess` => `RewriteBase /app/webroot/`

Make sure to place the RewriteBase directive just after `RewriteEngine on`.

## Images/CSS Broken

Usually this is a result of `.htaccess` files not being parsed. Make sure that the the directives that cover your li3 installation include the following line:

```apache
AllowOverride all
```

## Unexpected Character Input

If you get an error that looks like this:

```text
Warning: Unexpected character in input: '\' (ASCII=92) state=1 in /path/to/lithium/app/webroot/index.php on line 22
```

This means you're not running PHP 5.3 or later.  Please check your PHP version and update as appropriate.
