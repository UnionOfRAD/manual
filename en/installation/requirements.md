# Requirements

You need a **web server** to run li3, preferably running on your local machine (you do all of your development locally, right, right??). As PHP, the framework itself runs just fine on most popular web servers. [Setup guides are available for Apache, IIS, Lighttpd, and NGINX](./web-serves.md). 
 
* Web server (Apache, ISS, Lighttpd, NGINX)

Because the framework takes advantage of advanced **language** features, the minimum version requirement is PHP 5.3, although at least PHP 5.4 is recommended.

 * PHP 5.3+ is required and PHP 5.4+ is recommended 

<div class="note">
	We don't support certain features as we consider them broken, experimental or a hack:
	<ul>
	 <li>Magic Quotes must be disabled.
	 <li>Register Globals must be disabled.
	 <li>Function overloading must be disabled when using the `mbstring` extension.
	 <li>PHP should not be compiled with curlwrappers.
	 <li>Short Open Tags should be disabled. Not a strict requirement.
	</ul>
</div>

Applications often feature some sort of **data store**. As such, you may want to track down one of the following as well:

 * MongoDB
 * MySQL or MariaDB
 * PostgreSQL
 * SQLite
 * CouchDB


While not absolutely essential, a working knowledge of the [Git **version control system**](http://git-scm.com/) is useful for most aspects of development in general, and li3 specifically i.e. the workflow for contributing to projects is Git-based.

 * Git

Also not required for working with the framework itself, however it provides many useful tools for automating complex or repetitive tasks.

 * Command-line terminal

For **innovation and collaboration**:

 * Passion!
