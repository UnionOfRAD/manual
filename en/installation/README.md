# Installation

## Starting a New Project

The best way to start a project is to base it on a already available project distribution. There are distributions for general web projects or for projects that require an micro-framework style approach. 
Distributions come with a predefined app directory structure, some boilerplate code and the bundled lithium core library.

We'll base our new project off the officially supported [framework distribution](https://github.com/UnionOfRAD/framework). 

For the start we'll use GIT to clone the distribution's repository into 
the current directory as `project`. 

```bash
cd project
git submodule init
git submodule update
```

We'll than switch into that directory and initialize the submodules of that repository. Submodules are used in order to have no dependencies other than on the core library itself.

```bash
cd project
git submodule update --init
```

If everything worked as expected, you should now have the lithium core inside `project/libraries/lithium`. 

## Pedal to the Metal

For the purposes of this guide, we'll be using PHP's builtin development webserver. This is good for development purposes but should of course not be used in production. Instructions on how to use other
webservers are described at the end of this guide.

### Configuring PHP

The vanilla PHP configuration should be fine. However its always good to double check that certain configuration options are set correctly. Certain features are not supported as we consider those
broken, very experimental or a hack.

Verify in your php.ini that:

- Magic Quotes are disabled.
- Register Globals are disabled.
- Function overloading is disabled when using the `mbstring` extension.
- PHP should isn't compiled with curlwrappers.
- Short Open Tags are disabled. Although this is not a strict requirement.

While you're making PHP configuration changes, you might also consider having PHP display errors temporarily during development. Just change the relevant lines in your `php.ini`:

```ini
error_reporting = E_ALL
display_errors  = On
```

### Permissions

The framework will need write access to the `app/resources/tmp` as it used to store cached versions of 
already compiled templates or logs. The webserver use must be able to write to this directory.

### Starting the Webserver

Make sure you are in the root of the project. Now start PHP's builtin webserver using the following command. Your project is then being served at 127.0.0.1 on port 8080. 

```bash
php -S 127.0.0.1:8080 -t app/webroot index.php
```

Finally, pull up the project in your browser and visit [`http://127.0.0.1:8080`](http://127.0.0.1:8080).
At this point, you should be presented with the default home page. **You're up and running!**

<div class="note note-hint">
	Read more about setting up production webservers on the <a href="./servers.md">Servers</a> page.
</div>

## One or Two More Things

### Setup up the `li3` Command

Lastly, you'll want to set up the `li3` command so it's easy to use as you move around your filesystem. The `li3` command assists in tasks like code generation, documentation, and testing.

To do so, add the li3's console library directory to your shell's path. For our example above, and assuming you're using BASH, you'd add something like the following to your `~/.bash_profile` file:

```bash
PATH+=:/path/to/project/libraries/lithium/console
```

Once this has been done, you can execute the li3 command inside the app folder of any li3 app you have on your filesystem. If it's running successfully, you should get the following default usage output:

```text
USAGE
	li3 COMMAND [ARGS]

COMMANDS
...
```

### Getting the Most Recent Version

The methods described in the previous sections will download the most recent tagged version of
the framework and core library. In some cases, it may be desirable to update both to the very
latest available revision, which may not have been tagged yet.

```bash
cd project
git checkout dev

cd libraries/lithium
git checkout dev
```


