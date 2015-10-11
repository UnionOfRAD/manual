# Bootstrapping

Configuration in li3 is meant to be minimal and easy to understand. As such, it's based on a simple set of PHP files that are included as needed. The main boostrap file is `app/config/bootstrap.php` and houses all of the configuration for your application. While there might be a tendency to add bits of configuration to this file, realize it's really only meant as a switchhouse for other configuration details.

The core configuration sets are either actively required from this main file, or commented out (because we like to run lean by default). Before creating new files, peruse the configurations in `app/config/bootstrap`, as many commonly used mechanisms (caching, sessions, globalization) already have configuration examples in this folder. 

While we've included a lot of examples to get you started, feel free to add new files to the `boostrap` directory and require them from the main `bootstrap.php` file. If you're working in the cloud or interacting with a specific set of APIs or just need a place to keep a few global constants, those configurations are well-placed in this organization scheme.

### Action

The `action.php` configuration file is used to house request lifecycle filters. If you've got some piece of logic that sits in between lifecycle events, this is a great place for it. For starters, this file contains a filter that detects the current environment settings and loads all the routes in the main application and in all enabled plugins.

### Cache

This configuration file is used to set cache adapter and strategy settings. As your application matures, caching will become more important and more complex. Use this file to tell the application what storage adapter to use, along with the relevant settings. 

### Connections

The `connections.php` file houses persistent storage options. Most commonly, this means database connection details. 

### Console

Much like `action.php`, this file houses CLI-related filters and settings. If your console applications require specific settings due to your terminal emulation or color settings, this is the place for it. This file also contains some basic environmental and routes-based settings, but for console applications.

### Errors

Error and exception handling related configuration is kept here. For starters, you'll find a filter in this file that tells li3's ErrorHandler class how to trap exceptions and errors triggered in controller actions.

### G11n

All globalization configuration happens here. Timezone, current locale, catalog details and translation-related filters can already be found in this file. 

### Libraries

Every collection of classes located in a single base directory is handled as a _library_ in li3. With the exception of the primary library (initially called "app"), these packages of code are usually found in `/libraries` or `app/libraries` and are managed, listed, and enabled here.

The first part of this configuration file sets up the main path components the rest of li3 will base itself from. Unless you're doing some customization, you can probably leave this as-is. It actually keeps things running a bit faster than they would otherwise. 

At the bottom of the file, however, you'll see the main components of the appliation being registered by the `Libraries::add()` method. Note that the "app" library, including li3 itself are handled just like any other third-party addon or plugin.

This manual is also a library: enabled by downloading the codebase to `/libraries` (or `app/libraries`) and adding this to `libraries.php`:

```php
Libraries::add('manual');
```

### Media

The `Media` class is central to understanding how li3 handles and renders any given response. The `media.php` file handles the configuration surrounding this class. 

For instance, you can register different format handlers, or uncomment a filter that allows plugin static assets to be routed to more easily.

### Session

As titled, this configuration file handles the details surrounding user sessions. Details like adapter type, session name, and authentication configuration are all managed here.
