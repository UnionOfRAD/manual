# Deploying to Orchestra

## About Orchestra
Orchestra.io is a cloud-based deploying plattform for PHP applications. You can find out more at their website . 
Orchestra provides a "one-click" deployment out of a repository (can be either public or private) and a basic account is free 
(see [`Pricing`](http://www.engineyard.com/products/orchestra/pricing)). The Orchestra team always had Lithium in mind when desigin their 
services and you can always find one of them idling around in our #li3 freenode channel.

Before you can deploy your Lithium application, you first have to create an Orchestra account. Head over to their [`Signup`](http://www.engineyard.com/orchestra_signup) 
page and get yourself an account as it's free.

## Deploying a vanilla app
To get a feeling about how Orchestra works, you can deploy a vanilla Lithium app to your free account at no cost. As Orchestra supports git submodules and loads 
them automatically, we can deploy from the official `framework` repository and see how far we can get.

Login to your application and click on the `Deploy a free App` button to your right. Now fill out the form like this:

 - **Name**: unique-name-of-your-app
 - **Repo URL**: https://github.com/UnionOfRAD/framework.git
 - **Index File**: app/webroot/index.php

Now click on the `Launch App` button and wait while Orchestra deploys your application. When it's finished you can visit your application with the name you've provided 
earlier (http://unique-name.orchestra.io). You should now see the friendly Lithium welcome screen which indicates that everything has worked as expected.

## Next steps
Orchestra supports a variety of databases and modules out of the box. If you have further questions, visit their [`Knowledge Base`](http://docs.orchestra.io/kb) or 
catch them on IRC.