---
layout: post
title:  "Deploy Ember Application on Linode"
date:   2017-08-17 6:11:15
author: shubham
categories: Technical Ember
---

Welcome to the world of web applications.Now a days the frontend does not only consists of views and logic contained in javascripts, there is more to that. For example ember uses router, routes, controller, components, templates, views etc and encourages us to use the patterns which are good for maintainability and scalability. Apart from all this now we have to build the frontend app. So simple copy pasting of raw files does not work for deployment.

#### Building Ember application 
For building an ember application we just need to run a command in root folder of our application.
 ``` ember build ```
 You can also build an ember application for a specific environment like production or development by specifying a flag
 ``` ember build —environment=production```
 That’s it, your ember application is built and all the generated files are in the **dist** folder in the root folder of your application.

 #### Deploying 
 For deploying application on Linode (Apache server) we need to copy **dist** folder on server. For this we will use **scp**.
 But there might me two or more websites hosted on a single server so first we will create a folder with the name of our website in /var/www/html.

For this we need to ssh the server. Open a new tab in your terminal and enter - 
```ssh user-name@ip_address``` 
(Replace the user-name with your username and ip_address with the ip address of the server.)
This will prompt you for the password, enter your password and now we are ready to make changes in our server.

First we will create the folder in which our website will be copied.
```sudo mkdir /var/www/html/example.com```
(Replace example.com with your website name.)

Now we need to create 3 folders in this example.com for storing our website, for logs and backups.

{% highlight java %}
sudo mkdir -p /var/www/html/example.com/public_html
sudo mkdir -p /var/www/html/example.com/log
sudo mkdir -p /var/www/html/example.com/backups
{% endhighlight %}

Now we have to copy our **dist** folder from local into **example.com/public_html** folder on server. Run the below command locally.

```scp -r dist/* user-name@ip_address:/var/www/html/example.com/public_html ```

If this works for you, great but if you are getting the error that **permission denied** , don't panic. It is because you do not have the permission to **scp** into that folder. But you can **scp** into the root folder of your server.

First create a folder in the root of your server

```sudo mkdir public_html```

Now go back to your local directory in terminal and  scp into this public_html folder
```scp -r dist/* user-name@ip_address:public_html```

Copy the content of this public_html folder to  /var/www/html/example.com/public_html

```sudo cp -r public_html /var/www/html/example.com```

Next we need to configure our server so that it can redirect to our website whenever the ip address of the server or DNS associated with it is hit.

On an Apache server, the rewrite engine (mod-rewrite) must be enabled in order for Ember routing to work properly. If you upload your dist folder, going to your main URL works, but when you try to go to a route such as '{main URL}/example' and it returns 404, your server has not been configured for "friendly" URLs.
To fix this add a file called '.htaccess' to the root folder of your website. 

```sudo vim /var/www/html/example.com/public_html/.htaccess```

Add these lines:

{% highlight java %}
<IfModule mod_reqrite.c>
RewriteEngine On
RewriteRule ^index\.html$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*) index.html [L]
</IfModule>
{% endhighlight %}

Then create a config file example.com.conf in  ~/etc/apache2/sites-available (and obviously replace example.com with your website name).

```sudo vim  /etc/apache2/sites-available/example.com.conf```

Vim editor will open and you can edit it by typing “i”. Copy the below text

{% highlight java %}
# domain: example.com
# public: /var/www/html/example.com/public_html/

<VirtualHost *:80>
  # Admin email, Server Name (domain name), and any aliases
  ServerAdmin webmaster@example.com
  ServerName  example.com
  ServerAlias www.example.com

  # Index file and Document Root (where the public files are located)
  DirectoryIndex index.html index.php
  DocumentRoot /var/www/html/example.com/public_html
  # Log file locations
  LogLevel warn
  ErrorLog  /var/www/html/example.com/log/error.log
  CustomLog /var/www/html/example.com/log/access.log combined
</VirtualHost>
{% endhighlight %}

and replace example.com with your website name. Save and quit the vim editor by pressing **esc** and typing :wq and enter.

Enable the new website
```sudo a2ensite /etc/apache2/sites-available/example.com.conf```

Next we need to add few lines to ~/etc/apache2/apache2.conf so that server can detect our .htaccess file in the root folder of our website.

Open the config file
```sudo vim ~/etc/apache2/apache2.conf```
and add these lines

{% highlight java %}
<Directory /var/www/html/example.com>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
{% highlight java %}

Everything is set we just need to restart the server and associate the ip address of your server with the domain name that you can do using 

Restart the server
 ```sudo service apache2 restart```

Congratulations on deploying your first ember website.












