# Requirements

## Web Server

You need a **web server** to run your app, preferably running on your local machine (you do
all of your development locally, right, right??). As PHP, the framework itself runs just
fine on most popular web servers. [Setup guides are available for Apache, IIS, Lighttpd, and
NGINX](./web-servers.md).
 
## PHP 

Because the framework takes advantage of advanced **language** features, a recent PHP version
is required. The compatibility table below shows which framework version requires which PHP
version.

|              | required PHP    | recommended PHP | compatible PHP       |
| ------------ | --------------- | --------------- | -------------------- |
| **1.0.x**    | >= 5.3.6        | >= 5.4.0        | >= 5.3.6 \| < 5.7.0  |
| **1.1.x**    | >= 5.5.0        | >= 5.6.0        | >= 5.5.0             |

The vanilla PHP configuration should be in general fine. However its always good to double
check that certain configuration options are set correctly. Certain features are not supported
as we consider those broken, very experimental or a hack.

Please verify that:

- Magic Quotes are disabled.
- Register Globals are disabled.
- Function overloading is disabled when using the `mbstring` extension.
- PHP isn't compiled with curlwrappers.
- Short Open Tags are disabled. Although this is not a strict requirement.

While you're making PHP configuration changes, you might also consider having PHP display errors temporarily during development. Just change the relevant lines in your `php.ini`:

```ini
; Show me teh errors.
display_errors = On

; Either choose to see all errors or all, but no deprecation warnings.
error_reporting = E_ALL
; error_reporting = E_ALL & ~E_DEPRECATED
```

## Data Store

Applications often feature some sort of **data store**. As such, you may want to track down one
of the following as well:

 * MongoDB
 * MySQL or MariaDB
 * PostgreSQL
 * SQLite
 * CouchDB

## Version Control System

While not absolutely essential, a working knowledge of the [Git **version control
system**](http://git-scm.com/) is useful for most aspects of development in general, and li3
specifically i.e. the workflow for contributing to projects is Git-based.

## Command-line Terminal

Also not required for working with the framework, however it provides many useful tools
for automating complex or repetitive tasks.

## Passion!

... for **innovation and collaboration**.
