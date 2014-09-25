# li3 on Apache

## Basic Setup

In order to provide li3 applications with clean URLs, li3 ships with a set of `.htaccess` files for use with Apache, which will handle URL rewriting.

By default, these files can be utilized by finding all references to `AllowOverride` in your `httpd.conf` configuration file and setting the values to `All`. On an OS X system, your setup may look something like this:

```apache
<Directory "/Library/WebServer/Documents">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    Allow from all
</Directory>
```

Once you've made sure `AllowOverride` has been set correctly, you can toss a copy of li3 in your DocumentRoot. Using the past example, placing li3 in `/Library/WebServer/Documents/lithium` would allow you to access your application at http://localhost/lithium.

## Production Setup

In production, it is recommended that you set `AllowOverride` to `None` and instead, create a `<VirtualHost />` configuration pointed at your application's `webroot` directory, which contains the rewrite rules. For example, if your application is located in `/var/www/html/your_app`, your configuration would resemble the following:

```apache
<VirtualHost *:80>
	ServerName example.com
	DocumentRoot "/var/www/html/your_app/webroot"
	ErrorLog "logs/error_log"

	<Directory "/var/www/html/your_app/webroot">
		RewriteEngine On
		RewriteCond %{REQUEST_FILENAME} !-d
		RewriteCond %{REQUEST_FILENAME} !-f
		RewriteCond %{REQUEST_FILENAME} !favicon.ico$
		RewriteRule ^ index.php [QSA,L]
	</Directory>
</VirtualHost>
```

In this case, your app is available at http://example.com.
