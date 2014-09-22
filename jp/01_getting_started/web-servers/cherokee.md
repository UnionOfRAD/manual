# li3 on Cherokee

## About Cherokee
Cherokee is a web server build for speed and ease of configuration. It supports a variety of technologies, including PHP. If you're interested in using li3 with Cherokee, this guide is meant to cover the installation and configuration process to get you up and running quickly.

## Getting and running Cherokee

[Download ](http://www.cherokee-project.com/downloads.html) the appropriate package for your architecture.  Follow the instructions [here ](http://www.cherokee-project.com/doc/basics.html) to get your main instance configured and running.

## Virtual Server Creation

Refer to the Cherokee documentation on how to start an admin instance.  Access that instance (usually at http://localhost:9090/) and go to the 'Virtual Servers' section.  Click 'Wizards', 'Platform', 'Zend' and 'Run Wizard'.  Put the desired hostname (we'll use `lithium.local`) and set the path to `app/webroot` of your li3 installation.

![Screenshot](http://grab.by/2yty)

Click 'Submit'.

## Virtual Server Configuration

We'll need to reconfigure some of the settings that the Zend wizard uses to make things more li3-friendly. Open up your new virtual server and remove the 'index.html' default page.

![Screenshot](http://grab.by/2ytI)

Click on 'Host Match' and choose 'Wildcards'.  Add as many hostnames and/or aliases that you will use to access your lithium instance.

![Screenshot](http://grab.by/2ytQ)

Next, click on the 'Behavior' tab, and then the second rule with the type "Full Path".

![Screenshot](http://grab.by/2ytW)

Click on the 'Handler' tab, and change the handler to 'Redirection', set "show" to 'Internal', leave "Regular Expression" blank, but set "Substitution" to 'index.php'.

![Screenshot](http://grab.by/2yua)

Go back to the behaviors list (via clicking the name of your virtual server above), and this time we are going to change the 'Default' rule, and click on the 'Handler' tab.

![Screenshot](http://grab.by/2yul)

Change the following fields like so:

 * 'Regular Expression': ^([^\?]*)(\?(.+))?$
 * 'Subtitution': index.php?url=$1&$3

The rest of the settings are optional (and you really shouldn't change them until you get familiar with how Cherokee works).  Choose 'Graceful Restart' and click 'Save' under 'Save Changes'.

![Screenshot](http://grab.by/2yus)

## Conclusion

Open your browser and go to one of the hostnames you defined:

![Screenshot](http://grab.by/2yuI)

All done!