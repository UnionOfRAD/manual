# Lithium on Cherokee

## About Cherokee
Cherokee is a web server build for speed and ease of configuration. It supports a variety of technologies, including PHP. If you're interested in using Lithium with Cherokee, this guide is meant to cover the installation and configuration process to get you up and running quickly.

## Getting and running Cherokee

[Download ](http://www.cherokee-project.com/downloads.html) the appropriate package for your architecture.  Follow [the instructions here](http://www.cherokee-project.com/doc/basics.html) to get your main instance configured and running.

## Virtual Server Creation

Refer to the Cherokee documentation on how to start an admin instance.  Access that instance (usually at `http://localhost:9090/`) and go to the **Virtual Servers** section.  Click **Wizards**, **Platform**, **Zend** and **Run Wizard**.  Put the desired hostname (we'll use `lithium.local`) and set the path to `app/webroot` of your Lithium installation.

![Screenshot](http://tardis1.tinygrab.com/grabs/2c69aa4dc460487f0ffff61f2af019eb.png)

Click **Submit**.

## Virtual Server Configuration

We'll need to reconfigure some of the settings that the Zend wizard uses to make things more Lithium-friendly. Open up your new virtual server and remove the `index.html` default page.

![Screenshot](http://tardis1.tinygrab.com/grabs/b1b85bfb950820cc789b516806a3e27a.png)

Click on **Host Match** and choose **Wildcards**.  Add as many hostnames and/or aliases that you will use to access your lithium instance.

![Screenshot](http://tardis1.tinygrab.com/grabs/810e1a670fb5fe33ffe520f4f8b88678.png)

Next, click on the **Behavior** tab, and then the second rule with the type `Full Path`.

![Screenshot](http://tardis1.tinygrab.com/grabs/9494f6d409fd0bff894e491adeed7104.png)

Click on the **Handler** tab, and change the handler to **Redirection**, set **Type** to **Internal**, leave **Regular Expression** blank, but set **Substitution** to `index.php`.

![Screenshot](http://tardis1.tinygrab.com/grabs/394b1b644380c46e501b626589f60c9b.png)

Go back to the behaviors list (via clicking the name of your virtual server above), and this time we are going to change the **Default** rule, and click on the **Handler** tab.

![Screenshot](http://tardis1.tinygrab.com/grabs/b7c7bff8093bfb5a0d757c2f5f10d4ba.png)

Change the following fields like so:

 - **Regular Expression**: `^([^\?]*)(\?(.+))?$`
 - **Subtitution**: `index.php?url=$1&$3`

The rest of the settings are optional (and you really shouldn't change them until you get familiar with how Cherokee works). Choose **Graceful Restart** and click **Save** under **Save Changes**.

![Screenshot](http://tardis1.tinygrab.com/grabs/16caa6b887cb0ce8b02135c8b0553f9d.png)

## Conclusion

Open your browser and go to one of the hostnames you defined:

![Screenshot](http://tardis1.tinygrab.com/grabs/6789a487bc3a1550b3786d2a2acedc6d.png)

All done!