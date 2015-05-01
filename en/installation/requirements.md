# Requirements

## Web Server

You need a **web server** to run your app, preferably running on your local machine (you do
all of your development locally, right, right??). As PHP, the framework itself runs just
fine on most popular web servers. [Setup guides are available for Apache, IIS, Lighttpd, and
NGINX](./web-serves.md).
 
## PHP 

Because the framework takes advantage of advanced **language** features, a recent PHP version
is required. The compatibility table below shows which framework version requires which PHP
version.

|              | required PHP    | recommended PHP | compatible PHP       |
| ------------ | --------------- | --------------- | -------------------- |
| **1.0.x**    | >= 5.3.6        | >= 5.4.0        | >= 5.3.6 \| < 5.7.0  |
| **1.1.x**    | >= 5.5.0        | >= 5.6.0        | >= 5.5.0             |

We don't support certain features as we consider them broken, experimental or a hack:

* Magic Quotes must be disabled.
* Register Globals must be disabled.
* Function overloading must be disabled when using the `mbstring` extension.
* PHP should not be compiled with curlwrappers.
* Short Open Tags should be disabled. Not a strict requirement.

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
