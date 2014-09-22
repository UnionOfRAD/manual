# Console Applications

Console applications in li3 allow you to access your application
infrastructure from the command line. The `Console` libraries li3
features also allow you to perform oft-used features in a shell
environment.

# The `li3` Shell Command

Before we get started, you'll want to make sure that your shell knows
where the `li3` command is.

If my li3 installation was at `/usr/local/lithium`, I'd add the
following to my Bash configuration to make sure the `li3` command was
universally available to me:

```
PATH=$PATH:/usr/local/lithium/console
```

# Creating a new Command

User-defined commands are located in `/app/extensions/command/` by
default. Let's create a simple application that list the repositories of
a given organization.

First, create the new file at `/app/extensions/command/Repos.php`. This
is what we'll start with:

```php
namespace app\extensions\command;

class Repos extends \lithium\console\Command {}
```

If you run `$ li3` now, you'll see that li3 can already see your
command. Look towards the end of the output, just after "COMMANDS via
app."

```
COMMANDS via app
    repos
```

If you document the Repos class with a comment, it will be shown when
li3 is executed without any arguments.

# Command I/O

By default, the `run()` method of your newly created Command is called
when a user executes `li3 <command name>` from a shell. The easiest way
to get input from a user is via command-line arguments. Arguments passed
to the Command are supplied as arguments to `run()`.

```php
namespace app\extensions\command;

class Repos extends \lithium\console\Command {
  
  public function run($org = '') {
    echo "Org: $org\n";
  }
}
```

You can also use `in()` to ask the user for input:

```php
namespace app\extensions\command;

class Repos extends \lithium\console\Command {
  public function run($org = '') {
    if($org == '') {
      $org = $this->in("Organization?");
    }
    echo "Org: $org\n";
  }
}
```

And rather than using `echo` to send output to the user, we can use
`out()` or `error()` to send to STDOUT and STDERR. Apart from adding
newlines automatically, these methods can also send colored output by using style tags.

```php
namespace app\extensions\command;

class Repos extends \lithium\console\Command {
  public function run($org = '') {
    if($org == '') {
      $this->error("{:red}No organization supplied.{:end}");
      $org = $this->in("Organization?");
    }
    $this->out("{:green}Org: $org{:end}");
  }
}
```

# Adding Functionality

Finally, let's add a bit of functionality to interact with the GitHub
API. Because we have full access to li3 classes, we declare them via
`uses` above the class definition like we normally would and use those
classes in our Command logic:

```php
namespace app\extensions\command;

use lithium\net\http\Service;

class Repos extends \lithium\console\Command {
  public function run($org = '') {
    if($org == '') {
      $this->error("{:red}No organization supplied.{:end}");
      $org = $this->in("Organization?");
    }
    $this->out("{:green}Org: $org{:end}");

    $service = new Service(array(
      'scheme' => 'https',
      'host'   => 'api.github.com'
    ));

    $repos = $service->get('/orgs/' . $org . '/repos');
    $this->header("$org Repos:");
    foreach($repos as $repo) {
      $this->out($repo['full_name']);
    }
  }
}
```

Here's a sample of the output:

```
$ li3 repos UnionOfRad
Org: UnionOfRad
/-----------------
UnionOfRad Repos:
/-----------------
UnionOfRAD/phpca
UnionOfRAD/li3_sqlsrv
UnionOfRAD/framework
UnionOfRAD/li3_docs
UnionOfRAD/lithium_bin
UnionOfRAD/li3_design
UnionOfRAD/manual
UnionOfRAD/lithium
UnionOfRAD/li3_qa
UnionOfRAD/li3_cldr
UnionOfRAD/li3_lab
UnionOfRAD/li3_bot
UnionOfRAD/li3_lldr
UnionOfRAD/li3_quality
UnionOfRAD/li3_queue
UnionOfRAD/sphere
UnionOfRAD/li3_couchbase
UnionOfRAD/li3_sqltools
UnionOfRAD/li3_fixtures
```
