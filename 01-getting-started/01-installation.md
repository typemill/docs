# Installation

The installation of Typemill is as simple as this: 

* Go to [typemill.net](https://www.typemill.net) and download the latest zip-folder.
* Unzip the files, and upload them to your web-server.
* Go to `www.your-typemill-website.com/setup` and create an initial user.
* Login and configure your system, your themes, and your plugins. 

Don't forget to make these folders and files writable (set permission to `774`):

- `\cache`
- `\settings`
- `\content`
- `\media`

All settings and users are stored in the folder `settings`. You can manually edit these files, but it is not recommended because it might crash the system if done wrong.

If you need more detailed instructions, please read on.

[TOC]

## Download

There are two ways to copy Typemill to your local computer:

1. Go to [typemill.net](https://www.typemill.net), download the zip-archive and unzip it.
2. **Or** use GitHub and Composer.

If you use GitHub, then you can find the repository of Typemill on [github.com/typemill/typemill](https://github.com/typemill/typemill). Just open the command line and type:

````
git clone "https://github.com/typemill/typemill.git"
````

The GitHub version of Typemill ships without libraries. You have to download all libraries with Composer, otherwise Typemill won't run. If you don't have Composer installed yet, head over to the [Composer website](https://getcomposer.org/), and install it. After that, open your command line, go to your Typemill folder and type:

````
composer update
````

The exact command might vary depending on your local composer installation and configuration. If you face any problems, please check the Composer documentation.

That's it!

## Permissions

The following three folders and all files and folders inside must be writable:

- `\cache`
- `\settings`
- `\content`
- `\media`

To make the folders and files writable, use your ftp programm, click (or right-click, depending on your software) on the folder, choose `permissions` and change the permission to `744`. Use the recursive permission for all containing files and folders. If `744` does not work, try `774`.

## htaccess 

If you run your website with HTTPS (recommended), or if you want to redirect www-URLs to non-www URLs, then please check the htaccess file in the root folder. There are several use cases already prepared, and you can simply uncomment them, if needed. 

## Run Locally

If you are a developer and if you want to run Typemill locally, then simply download Typemill (zip or git) and visit your local folder like `localhost/typemill`. If your server is configured correctly, then no additional work is required.

## Run on Linux/Ubuntu

Some users reported errors with Linux/Ubuntu. If you run Typemill locally on Linux, then you have to double check if mod-rewrite is configured correctly. If you face any problems, please follow the steps described in this article: https://hostadvice.com/how-to/how-to-enable-apache-mod_rewrite-on-an-ubuntu-18-04-vps-or-dedicated-server/

Enable the mod-rewrite modules:

````
    $ sudo a2enmod rewrite 
    $ sudo a2enmod actions 
````

Then open your virtual host file:

````
    $ sudo xed /etc/apache2/sites-available/000-default.conf
````

And add this configuration inside your virtual-host-tag: 

````
    <Directory /var/www/html>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Require all granted
    </Directory>
````

Finally, restart your server:

````
    $ sudo systemctl restart apache2
````

## Error Reporting

Typemill ships with error reporting off, because errors should never been displayed on a live system. But if you run Typemill locally, you can switch error reporting on. Simply open the file `settings.yaml` in the `settings` folder and add a new line:

````
displayErrorDetails: true
````

Don't forget to turn error reporting off again before you deploy your website live.

