# Creating a Lithium Plugin

If you've written a piece of functionality that you want to share among multiple applications, or [distribute within the Lithium community](http://lab.lithify.me/), you'll want to create a _plugin_. The good news is, if you've already created a Lithium application, you're more than half way there.

First, to understand how Lithium deals with plugins, it's important to understand how it deals with _libraries_, so make sure you've read "[Working with External Libraries](http://lithify.me/docs/manual/01_getting_started/external_libs)". In Lithium, everything is a library, including the Lithium core, other frameworks, your application, and of course, plugins. By convention, plugins are simply libraries that conform to the same organizational conventions as a Lithium application.

## Requirements

While there are very few (really no) _hard_ requirements for Lithium plugins, there are several very strong recommendations. First and foremost is that you follow the namespacing standards outlined in the [`Libraries` class documentation](http://lithify.me/docs/lithium/core/Libraries). Second, your plugin should include a `config` directory containing a `bootstrap.php` file. Finally, as mentioned above, it's best to conform to the same directory structure as a Lithium application. And of course, classes must be namespaced accordingly, with the root namespace matching the name of the plugin. (An exception to this would be if you were using an organizational or vendor-level namespace within which your plugin lived, in which case the plugin's root namespace would look like `vendor\plugin_name`).

## What Can Plugins Include?

Because Lithium treats everything as libraries, and because all libraries are essentially on the same playing field, a plugin can include anything. Any type of class or resource that could be included in the Lithium core or in an application can be included in a plugin. This includes models, controllers, helpers, adapters, generic classes, even custom routes and web assets.

Common examples of things to include in plugins are adapters for cache engines, databases or web services, or a collection of models to share between multiple applications that use the same data. Other plugins introduce new classes with new functionality which can have their own types of adapters, such as the [`li3_access` plugin](https://github.com/tmaiaroto/li3_access), which adds access control support. Some plugins, like [`li3_twig`](http://dev.lithify.me/li3_twig), [`li3_doctrine`](http://dev.lithify.me/li3_doctrine) and [`li3_pdf`](http://dev.lithify.me/li3_pdf), wrap other libraries to provide tight integration with Lithium.

## Creating Your First Plugin

You can create a plugin simply by running the following from any directory, substituting `my_plugin` for whatever you want to call it:

	li3 library extract plugin my_plugin

This creates a new plugin from Lithium's default plugin template. When completed, it should report back the following:

	my_plugin created in /current/directory from /.../lithium/console/command/create/template/plugin.phar.gz

## Installing and Configuring

As with other libraries, plugins are installed in your local or system `libraries` directory by cloning, symlinking, etc. (Again, see "[Working with External Libraries](http://lithify.me/docs/manual/01_getting_started/external_libs)"). An important thing to know when developing plugins is that when your library is registered through [`Libraries::add()`](http://lithify.me/docs/lithium/core/Libraries::add(), it can also be configured with custom settings.

While `add()`'s `$config` parameter specifies several default configuration settings, it is also possible to pass any other arbitrary information that you may wish to use in your plugin. This data can then be retrieved within your plugin by calling `Libraries::get('plugin_name')` or `Libraries::get('plugin_name', 'key')`.

## Routes and Web Assets

As mentioned above, your plugin can also include custom routing and web assets. By default, these are both based on filters that are registered in the primary application's bootstrap files. To handle routing, you'll find a default filter in `config/bootstrap/action.php`, which iterates over all registered libraries in reverse order, looking for `config/routes.php` files to include. While the application can easily customize this behavior, by default, simply including `config/routes.php` in your plugin will automatically register your custom routes with the application.

Likewise with static assets, such as JavaScript, images and CSS files, they are loaded from plugins using a filter which can be found in `config/bootstrap/media.php`. By default, this is commented out in the bootstrap of the primary application, so it should first be enabled. Then, assets can be loaded from the `webroot` directory of the plugin (this directory can be overridden by passing a custom value to the `'webroot'` key in `Libraries::add()`).

While this works simply in development, the performance is not as good as direct filesystem access. In production, it's better to create a symlink from `libraries/plugin_name/webroot` to `main_app/webroot/plugin_name`.