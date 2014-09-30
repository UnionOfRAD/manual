# Installation

## Getting the Code

This easiest way to get a fresh copy of li3 is by downloading an archive from GitHub. You can view and download versions here:

https://github.com/UnionOfRAD/lithium/releases

Under the hood, li3 is actually separated in two different repositories. One is called `framework` and the other one `lithium`. The `framework` repository holds everything you need to instantly bootstrap your application, while the `lithium` repository holds the li3 core. This way you can reuse the li3 core for other projects or just include some libraries if you need them. 

The repositories are hosted on GitHub, where you can also download tarballs if you just want to play around and not fetch updates through a managed repository. The normal process of fetching li3 by source is to clone the `framework` repository and then install `lithium` as a submodule (which is already configured for you).

```bash
git clone git://github.com/UnionOfRAD/framework.git project
cd project
git submodule init
git submodule update
```

If everything worked as expected, you should now have the lithium core inside `project/libraries/lithium`. If you've downloaded the tarballs, make sure to unpack the core in the correct directory.

## Getting the most recent revision (optional)

The method described in the previous section will download the most recent tagged version of li3. In some cases, it may be desirable to update li3 to the very latest available revision, which may not have been tagged yet.

```bash
cd libraries/lithium
git pull origin master
```

## Pedal to the Metal

For the purposes of this guide, we'll assume you're running Apache. Before starting things up, make sure mod_rewrite is enabled, and the AllowOverride directive is set to 'All' on the necessary directories involved. Be sure to restart the server before checking things.

Another quick thing to check is to make sure that magic quotes have been completely disabled in your PHP installation. If you're seeing an exception error message initially, you might have magic quotes enabled. For more information on disabling this feature, see the [PHP manual](http://www.php.net/manual/en/security.magicquotes.disabling.php).

While you're making PHP configuration changes, you might also consider having PHP display errors temporarily during development. Just change the relevant lines in your `php.ini`:

```ini
error_reporting = E_ALL
display_errors  = On
```

Finally, pull up li3 in your browser. For this example, we're running Apache locally. Assuming you have a default configuration, and you cloned li3 into your document root directory, you can visit [`http://localhost/project`](http://localhost/project).

At this point, you should be presented with the li3 default home page. You're up and running!

## One More Thing

Lastly, you'll want to set up the `li3` command so it's easy to use as you move around your filesystem. The `li3` command assists in tasks like code generation, documentation, and testing.

To do so, add the li3's console library directory to your shell's path. For our example above, and assuming you're using the bash shell, you'd add something like the following to your `~/.bash_profile` file:

```bash
PATH=$PATH:/path/to/docroot/lithium/libraries/lithium/console
```

Once this has been done, you can execute the li3 command inside the app folder of any li3 app you have on your filesystem. If it's running successfully, you should get the following default usage output:

```text
USAGE
	li3 COMMAND [ARGS]

COMMANDS
...
```
