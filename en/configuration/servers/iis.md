# li3 on Internet Information Services (IIS)

## About IIS

IIS Is a Microsoft-supplied product which manages many internet services—one of them being HTTPD—that comes standard on most Windows machines.

## Obtaining IIS

To install IIS under Windows you need to go to "Add/Remove Programs" in your control panel. After entering the "Add/Remove Programs" window, click "Turn Windows features on or off" and locate the following:

- Internet Information Services
- Internet Information Services Hostable Web Core

For a quick setup, check both. If you want to remove specific features, you can expand "Intenernet Information Services" and remove unwanted features.

Once this is done you can now locate IIS within your Start menu or find it through search by typing "IIS".

## Preparations

To make IIS work well with li3's pretty URL system, install the URL Rewrite extension located on the [http://www.iis.net/download/URLRewrite](iis.net website) and follow the proper installation instructions located there.

Next, follow the setup instructions for PHP on IIS - [http://php.iis.net/](http://php.iis.net/)

- IIS 6.0 - [http://learn.iis.net/page.aspx/247/using-fastcgi-to-host-php-applications-on-iis-60/](http://learn.iis.net/page.aspx/247/using-fastcgi-to-host-php-applications-on-iis-60/)<br />
- IIS 7.0 - [http://learn.iis.net/page.aspx/246/using-fastcgi-to-host-php-applications-on-iis-7/](http://learn.iis.net/page.aspx/246/using-fastcgi-to-host-php-applications-on-iis-7/)

Setup a site by right clicking "Sites" and click "Add Web Site"

1. Enter your website name.
2. Enter the physical path to your application's webroot directory (_\path\to\lithium\app\webroot_).
3. _Optional:_ If your content is in your home directory, click "Connect As" and click "Specific User". From therec enter your Windows login details, as this will allow IIS to have proper access to the application.
4. Enter the desired port.
5. Click "OK".

## Adding Rewrite Rules to IIS

1. Locate your site in the "Sites" tree and click it.
2. Locate the URL Rewrite icon from the extension installed earlier.
3. On the right hand side under "Inbound Rules" click "Import Rules"
4. Click the ellipsis (...) next to the field box and locate the `.htaccess` file within your `\path\to\lithium\app\webroot\` directory and select it.
5. Click "Import".
6. On the right hand side, click "Apply".

Your li3 application should now be accessible via IIS.

## Notes

Tested with IIS7 (which is shipped with Windows 7).
