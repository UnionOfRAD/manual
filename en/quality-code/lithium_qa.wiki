# QA Plugin for Lithium

## Requirements

Lithium QA can be used as library plugged into a Lithium application. 

Code in the master branch should always be compatible with the most recent Lithium core. However sometimes changes have to be made rendering QA incompatible with older cores. In that case please choose a tagged version of QA. Please note that components of QA will try to interact with the VCS of the project you are checking. Only GIT is supported a such.

## Installation

We'll first obtain a new Lithium application. Than register QA as a submodule into the applications libraries. 

{{{
git clone git://github.com/UnionOfRAD/framework.git new
cd new
git submodule add git://github.com/UnionOfRAD/li3_qa.git libraries/li3_qa
}}}

Now we make the application aware of the new library by adding the following line to `config/bootstrap/libraries.php`.

{{{
Libraries::add('li3_qa');
}}}

Finally we pull in all submodules in this case the application, its submodules (Lithium), li3_qa and li3_qa's submodules (phpca).

{{{
git submodule update --init --recursive
}}}

## Syntax Command

Files can be syntax checked using the Syntax command which comes with the application. The command utilizes [PHPca](http://github.com/UnionOfRAD/phpca/) to check the syntax of PHP files against a set of rules. These rules are based upon the [Lithium Coding Standards](https://github.com/UnionOfRAD/lithium/wiki/Spec%3A-Coding) and the [Lithium Code Documentation](https://github.com/UnionOfRAD/lithium/wiki/Spec%3A-Documenting) Standards.

The basic usage is: 

{{{
li3 syntax [--metrics] [--blame] PATH
}}}

Here we are checking the whole file tree of our pet project:

{{{
li3 syntax /path/to/project
}}}

If you want to log the output to a file (and your on *nix) this might come in handy:

{{{
li3 syntax /path/to/project 2>&1 | tee verify_project.log
}}}

Blame each failure and show metrics:

{{{
li3 syntax --metrics --blame /path/to/project
}}}

## GIT Pre Commit Hook

This pre commit hook is based upon the example found in `.git/hooks/pre-commit.sample`. Copy the sample script to `/path/to/project/.git/hooks/pre-commit` and make it executable. Then, replace the code in the script with the code shown below and adjust the paths to Lithium QA and the li3 command.

{{{
cd /path/to/project
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit
chmod a+x .git/hooks/pre-commit
}}}
   
Now add the following code to .git/hooks/pre-commit and adjust the `LI3` value.

{{{
#!/bin/sh

LI3=/path/to/lithium/libraries/lithium/console/li3

if git-rev-parse --verify HEAD >/dev/null 2>&1
then
    AGAINST=HEAD
else
    # Initial commit: diff against an empty tree object
    AGAINST=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

EXIT_STATUS=0
PROJECT=`pwd`

for FILE in `git diff-index --cached --name-only --diff-filter=AM ${AGAINST}`
do
    ${LI3} syntax ${PROJECT}/${FILE}
    test $? != 0 && EXIT_STATUS=1
done

exit ${EXIT_STATUS}
}}}

Now when committing each file the syntax is checked. The commit is aborted if a check failed. If you don't want to have the hook run on commit pass the `--no-verify` option to git commit.

## Covered Command

Allows for checking if classes in a given library (or project) all have corresponding tests according to the Lithium standard. Also prints out coverage percentage.
{{{
li3 covered /path/to/library
}}}

Will output something similar to the following.

{{{
has test |  95.00% | lithium\util\Collection
has test |  97.98% | lithium\util\Inflector
has test | 100.00% | lithium\util\collection\Filters
has test |     n/a | lithium\test\Controller
has test |  95.24% | lithium\test\Dispatcher
 no test |     n/a | lithium\test\Filter
}}}

