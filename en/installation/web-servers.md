# Web Servers

## Using Apache httpd

Before starting things up, make sure `mod_rewrite` is enabled, and the `AllowOverride` directive
is set to `All` on the necessary directories involved. Be sure to restart the server before
checking things.

In order to provide applications with clean URLs, distributions ship with a set of `.htaccess` files for use with Apache, which will handle URL rewriting.

By default, these files can be utilized by finding all references to `AllowOverride` in your `httpd.conf` configuration file and setting the values to `All`. On an OS X system, your setup may look something like this:

```apache
<Directory "/Library/WebServer/Documents">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    Allow from all
</Directory>
```

Once you've made sure `AllowOverride` has been set correctly, you can toss a project in your `DocumentRoot`. Using the past example, placing the project in `/Library/WebServer/Documents/project` would allow you to access your application at `http://localhost/project`.

**In production**, it is recommended that you set `AllowOverride` to `None` and instead, create a `<VirtualHost />` configuration pointed at your application's `webroot` directory, which contains the rewrite rules. For example, if your application is located in `/var/www/html/project`, your configuration would resemble the following:

```apache
<VirtualHost *:80>
	ServerName example.com
	DocumentRoot "/var/www/html/project/app/webroot"
	ErrorLog "logs/error_log"

	<Directory "/var/www/html/project/app/webroot">
		RewriteEngine On
		RewriteCond %{REQUEST_FILENAME} !-d
		RewriteCond %{REQUEST_FILENAME} !-f
		RewriteCond %{REQUEST_FILENAME} !favicon.ico$
		RewriteRule ^ index.php [QSA,L]
	</Directory>
</VirtualHost>
```

In this case, your app is available at `http://example.com`.

## Using NGINX

Nginx requires a simple rewrite configuration that enables it to serve dynamic URLs. With nginx, each domain has its own configuration file that encapsulates all such rewriting rules.

The following example file is typically stored at `/etc/nginx/sites-available/example.org.conf`. All instances of `example.org` can be replaced with the domain name of your site or application.

```nginx
server {
	listen 80;
	server_name example.org;
	root /path/to/project/app/webroot;
	
	index index.php;
	try_files $uri $uri/ /index.php?$args;	

	location ~ \.php$ {
		# These are example paths. Check your FPM configuration as your setup may vary.
		include /usr/local/etc/nginx/fastcgi.conf;
		
		# Either use a FPM socket...
		fastcgi_pass unix:/usr/local/var/run/php-fpm.socket;
        
		# ... or an address to bind to.
		# fastcgi_pass 127.0.0.1:9000;
	}
}
```

## Using Internet Information Services (IIS)

IIS Is a Microsoft-supplied product which manages many internet services—one of them being HTTPD—that comes standard on most Windows machines.

### Obtaining IIS

To install IIS under Windows you need to go to "Add/Remove Programs" in your control panel. After entering the "Add/Remove Programs" window, click "Turn Windows features on or off" and locate the following:

- Internet Information Services
- Internet Information Services Hostable Web Core

For a quick setup, check both. If you want to remove specific features, you can expand "Intenernet Information Services" and remove unwanted features.

Once this is done you can now locate IIS within your Start menu or find it through search by typing "IIS".

### Preparations

To make IIS work well with li3's pretty URL system, install the URL Rewrite extension located on the [iis.net website](http://www.iis.net/download/URLRewrite) and follow the proper installation instructions located there.

Next, follow the setup instructions for [PHP on IIS](http://php.iis.net/) for either [IIS 6.0](http://learn.iis.net/page.aspx/247/using-fastcgi-to-host-php-applications-on-iis-60/) or [IIS 7.0](http://learn.iis.net/page.aspx/246/using-fastcgi-to-host-php-applications-on-iis-7/).

Setup a site by right clicking "Sites" and click "Add Web Site"

1. Enter your website name.
2. Enter the physical path to your application's webroot directory (_\path\to\lithium\app\webroot_).
3. _Optional:_ If your content is in your home directory, click "Connect As" and click "Specific User". From therec enter your Windows login details, as this will allow IIS to have proper access to the application.
4. Enter the desired port.
5. Click "OK".

### Adding Rewrite Rules to IIS

1. Locate your site in the "Sites" tree and click it.
2. Locate the URL Rewrite icon from the extension installed earlier.
3. On the right hand side under "Inbound Rules" click "Import Rules"
4. Click the ellipsis (...) next to the field box and locate the `.htaccess` file within your `\path\to\lithium\app\webroot\` directory and select it.
5. Click "Import".
6. On the right hand side, click "Apply".

Your li3 application should now be accessible via IIS.

<div class="note">
	Tested with IIS7 (which is shipped with Windows 7).
</div>

## On Lighty

This quick guide is meant to help Lighty users get a development setup running in order to create li3 apps served up by Lighttpd. Lighttpd is a web server designed and optimized for high-performance production environments. It has a small memory footprint, yet contains a rich feature set. If you're planning on using li3 and Lighttpd together, this guide is for you.

### Running Lighty

You can download a copy of Lighty on the [official website](http://www.lighttpd.net/download/). Once downloaded, follow the instructions to get a running instance.

Installation details should be found in the `INSTALL` file packaged with the downloaded archive. Online installation tips can also be found on [the Lighttpd website](http://redmine.lighttpd.net/projects/lighttpd/wiki/InstallFromSource).

### Configuration

Let us assume for this example that you're developing locally and would like your app to be accessible at `http://lithium.local/li3`.

First, enable `mod_magnet` in your `lighttpd.conf`.

```lighty
server.modules += ( "mod_magnet" )
```

Then, save the following script in a file named `li3.lua`, preferably somewhere near your `lighttpd.conf`.

```lighty
	-- Helper function
	function file_exists(path)
	  local attr = lighty.stat(path)
	  if (attr) then
	      return true
	  else
	      return false
	  end
	end
	function removePrefix(str, prefix)
	  return str:sub(1,#prefix+1) == prefix.."/" and str:sub(#prefix+2)
	end

	--[[
	  Prefix without the trailing slash.
	  If you are _not_ serving out of a subfolder, leave `prefix` empty.
	--]]
	local prefix = '/li3'

	-- The magic ;)
	if (not file_exists(lighty.env["physical.path"])) then
	    -- File still missing: pass it to the fastCGI backend.
	    request_uri = removePrefix(lighty.env["uri.path"], prefix)
	    if request_uri then
	      lighty.env["uri.path"]          = prefix .. "/index.php"
	      local uriquery = lighty.env["uri.query"] or ""
	      lighty.env["uri.query"] = uriquery .. (uriquery ~= "" and "&" or "") .. "url=" .. request_uri
	      lighty.env["physical.rel-path"] = lighty.env["uri.path"]
	      lighty.env["request.orig-uri"]  = lighty.env["request.uri"]
	      lighty.env["physical.path"]     = lighty.env["physical.doc-root"] .. lighty.env["physical.rel-path"]
	    end
	end
	-- Fallthrough will put it back into the lighty request loop..
	-- That means we get the 304 handling for free. ;)
```

Finally, in your `lighttpd.conf`, add the following conditional:

```lighty
	$HTTP["host"] =~ "lithium.local" {
		server.document-root = "/path/to/your/app/webroot/"
		magnet.attract-physical-path-to = ( "/path/to/li3.lua" )
	}
```

You'll probably need to add a line item in your `/etc/hosts` file as well:

```text
127.0.0.1 lithium.local
```

Restart your lighttpd process, and you're done!
