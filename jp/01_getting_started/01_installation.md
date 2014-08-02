# Installing Lithium

## Getting the Code

This easiest way to get a fresh copy of Lithium is by downloading an archive from our website. You can view and download versions here:

	http://dev.li3.me/lithium/versions

Li3's source is also housed in a Git repo. If you'd like to clone a copy, start by creating a new account at [http://dev.li3.me/users/add](http://dev.li3.me/users/add). Once you've successfully created your new account, add your SSH key at [http://dev.li3.me/users/account](http://dev.li3.me/users/account). If you've never completed that process before, sometimes its as simple as typing in 'ssh-keygen' in a console.

	$ ssh-keygen

	Generating public/private rsa key pair.
	Enter file in which to save the key (/Users/lithium/.ssh/id_rsa):
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /Users/lithium/.ssh/id_rsa.
	Your public key has been saved in /Users/lithium/.ssh/id_rsa.pub.

Once the public key has been created, copy and paste it's contents into the `Ssh Keys` section of your account settings at dev.li3.me.

Now that your key has been set up, clone yourself a copy of Li3 inside of your web server's document root:

	$ cd /path/to/docroot
	$ git clone code@dev.li3.me:lithium.git

	Initialize lithium/.git
	Initialized empty Git repository in /path/to/docroot/lithium/.git/
	Identity added: /Users/lithium/.ssh/id_rsa (/Users/lithium/.ssh/id_rsa)
	remote: Counting objects: 1638, done.
	remote: Compressing objects: 100% (1154/1154), done.
	remote: Total 1638 (delta 863), reused 509 (delta 226)
	Receiving objects: 100% (1638/1638), 387.74 KiB, done.
	Resolving deltas: 100% (863/863), done.

You'll be asked a password during this operation. Make sure to supply the same one you did when you created your SSH key.

## Advanced Setup

If you've got a system that's hosting many Lithium apps, sometimes it's beneficial to point a number of applications at the same set of core Lithium libraries. In this case, you'd want to place Lithium somewhere outside of your web server's document root.

For example, let's say we've got lithium installed in `/usr/local/lib/lithium/` and some Lithium applications at `/home/apps/first-app/` and `/home/apps/second-app`.

The two applications mentioned aren't complete copies of the Lithium codebase: they're just copies of the `app` folder (e.g. there should be a `/home/apps/first-app/config` folder).

First, you'll want to create two virtual hosts for the applications on the system. Once those are in place, each application's bootstrap will need to be informed of where the Lithium libraries are. Adjust both application's `/config/bootstrap.php` file like so:

```
define('LITHIUM_LIBRARY_PATH', '/usr/local/lib/lithium/libraries');
```

## Pedal to the Metal

For the purposes of this guide, we'll assume you're running Apache. Before starting things up, make sure mod_rewrite is enabled, and the AllowOverride directive is set to 'All' on the necessary directories involved. Be sure to restart the server before checking things.

Another quick thing to check is to make sure that magic quotes have been completely disabled in your PHP installation. If you're seeing an exception error message initially, you might have magic quotes enabled. For more information on disabling this feature, see the [PHP manual](http://www.php.net/manual/en/security.magicquotes.disabling.php).

While you're making PHP configuration changes, you might also consider having PHP display errors temporarily during development. Just change the relevant lines in your php.ini:

```
	error_reporting  =  E_ALL
	display_errors   =  true
```

Finally, pull up li3 in your browser. For this example, we're running Apache locally. Assuming you have a default configuration, and you cloned Lithium into your document root directory, you can visit [`http://localhost/lithium`](http://localhost/lithium).

At this point, you should be presented with the Li3 default home page. You're up and running!

## One More Thing

Lastly, you'll want to set up the li3 command so it's easy to use as you move around your filesystem. The li3 command assists in tasks like code generation, documentation, and testing.

To do so, add the Lithium's console library directory to your shell's path. For our example above, and assuming you're using the bash shell, you'd add something like the following to your `~/.bash_profile` file:

	PATH=$PATH:/path/to/docroot/lithium/libraries/lithium/console

Once this has been done, you can execute the li3 command inside the app folder of any Li3 app you have on your filesystem. If it's running successfully, you should get the following default usage output:

	USAGE
		li3 COMMAND [ARGS]

	COMMANDS
		create
			The `create` command allows you to rapidly develop your models, views, controllers, and tests
			by generating the minimum code necessary to test and run your application.

		g11n
			The `G11n` set of commands deals with the extraction and merging of
			message templates.

		test
			Runs a given set unit tests and outputs the results.

	See `li3 help COMMAND` for more information on a specific command.
