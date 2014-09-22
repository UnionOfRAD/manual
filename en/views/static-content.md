# Static Content
There may be cases where your application has static content that needs to be served. li3 provides out-of-the-box functionality to do this via the Pages controller.  Static content displayed using the pages controller should be stored in the `views/pages` folder of the application.

The application's default routing will automatically render static pages with no additional configuration necessary.  The default (`/`) route will render the `home` template.

Other static pages in the `views/pages` directory can be called by name from the URL.  As an example, pointing the browser to `/pages/about` would render the `views/pages/about.html.php` view, if the page exists.

It is also possible to organize and/or nest static pages into directories under `views/pages`.  For example, a file located at `views/pages/about/company.html.php` would be displayed by pointing the browser to `/pages/about/company`.
