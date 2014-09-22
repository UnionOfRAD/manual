# li3 Requirements

You need a web server to run li3, preferably running on your local machine (you do all of your development locally, right, right??). As PHP, li3 runs fine on most all popular web servers. Setup guides are available for Apache, Cherokee, Lighttpd, and Nginx. Because li3 takes advantage of advanced language features, the minimum version requirement is PHP 5.3.

 * Web server
 * PHP 5.3+

Applications built with li3 often feature some sort of data store. As such, you may want to track down one of the following as well:

 * MongoDB
 * MySQL
 * PostgreSQL
 * SQLite
 * CouchDB

li3 doesn't support certain features as it considers them broken, experimental or a hack:

 * Magic Quotes must be disabled.
 * Register Globals must be disabled.
 * Function overloading must be disabled when using the `mbstring` extension.
 * PHP should not be compiled with curlwrappers.
 * Short Open Tags should be disabled. Not a strict requirement.

While not absolutely essential, a working knowledge of the [Git version control system](http://git-scm.com/) is useful for most aspects of development in general, and li3 specifically. The li3 plugin installation/repository tools rely on Git, and the workflow for contributing to projects is Git-based.

 * Git

Also not required for working with li3 itself, however it provides many useful tools for automating complex or repetitive tasks.

 * Command-line terminal

For innovation and collaboration:

 * Passion!
