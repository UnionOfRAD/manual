# Deploying to the Cloud

## Engine Yard

Engine Yard is a cloud-based deploying plattform with decent support for PHP applications. You can find out more at their website. 
Engine Yard provides a "one-click" deployment out of a repository (can be either public or private) and a [basic account is free](https://www.engineyard.com/trial). The Engine Yard team always had li3 in mind when desigin their services and you can always find one of them idling around in our #li3 freenode channel.
Before you can deploy your li3 application, you first have to create an Engine Yard account. Head over to their Signup page and get yourself an account as it's free.

### Deploying a Vanilla App

To get a feeling about how Engine Yard works, you can deploy a vanilla li3 app to your free account at no cost. As Engine Yard supports git submodules and loads them automatically, we can deploy from the official framework repository and see how far we can get.

Login to your application and click on the `Deploy a free App` button to your right. Now fill out the form like this:

 - **Name**: unique-name-of-your-app
 - **Repo URL**: https://github.com/UnionOfRAD/framework.git
 - **Index File**: app/webroot/index.php

Now click on the `Launch App` button and wait while Orchestra deploys your application. When it's finished you can visit your application with the name you've provided 
earlier (http://unique-name.orchestra.io). You should now see the friendly li3 welcome screen which indicates that everything has worked as expected.

### Next Steps

Orchestra supports a variety of databases and modules out of the box. If you have further questions, visit their [`Knowledge Base`](http://docs.orchestra.io/kb) or 
catch them on IRC.
