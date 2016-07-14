# Static Code Analysis

Simply defined, static code analysis reviews and analyzes code outside of the runtime environment. In short, this means that the code is analyzed without actually running the code.  li3 offers a plugin that can perform certain types of static code analysis on your application.

## Installing

The _quality_ plugin for li3 is available at `git://github.com/UnionOfRAD/li3_quality.git`.  Once the repository is cloned into your `libraries` folder, It is activated by adding the following code to `config/bootstrap/libraries.php` file:

```php
Libraries::add('li3_quality');
```

## Available Tools

The following tools are available in `li3_quality` for performing static code analysis:

* Syntax: checks the conformance of your code against li3's coding standards
* Coverage: analyzes the percentage of code covered by unit tests
* Documented: looks for undocumented classes and methods in the library
* Complexity: measures cyclomatic complexity in the library

## Code Analysis from the Browser

The quality plugin is woven into li3's unit testing framework, and there are features of static code analysis that are available from li3's browser based interface, available by pointing the browser to `http://example.com/test`

Once you have made a choice on what is going to be tested from the left menu, options for measuring cyclomatic complexity, code coverage, and identifying syntax violations are available from the top menu.

## Code Analysis from the Command Line

`li3_quality` is also available for use from the command line:

```sh
li3 quality
```

```
li3 console started in the development environment. Use the --env=environment key to alter this.
USAGE
    li3 quality syntax
    li3 quality documented
    li3 quality coverage
```
