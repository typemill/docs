# Installation

The installation of TYPEMILL is as simple as that: 

- Go to [typemill.net](http://www.typemill.net) and download the TYPEMILL files.
- Unzip the files and upload them to your web-server.
- Go to `www.your-typemill-website.com/setup` and create an initial user.
- Login and configure your system, your themes and your plugins. 

Don't forget to make some folders and files writable (set permission to `774`):

- `\cache`
- `\settings`
- `\content`
- `\media`

All settings and users are stored in the folder `settings`. You can manually edit these files, but it is not recommended because it might crash the system if done wrong.

If you need more detailed instructions, please read on.

## Download

There are two ways to copy TYPEMILL to your local computer:

1. Go to [typemill.net](http://www.typemill.net), download the zip-archive and unzip it.
2. **Or** use GitHub and Composer.

If you use GitHub, then you can find the repository of TYPEMILL on [github.com/trendschau/typemill](https://github.com/trendschau/typemill). Just open the command line and type

````
git clone "https://github.com/trendschau/typemill.git"
````

The github-version of TYPEMILL ships without libraries. You have to download all libraries with composer, otherwise TYPEMILL won't run. If you don't have composer installed yet, head over to the [composer website](https://getcomposer.org/) and install it. After that, open your command line, go to your TYPEMILL folder and type:

````
composer update
````

The exact command might vary depending on your local composer installation and configuration. If you face any problems, please check the documentation of composer.

That's it!

## Permissions

The following three folders and all files and folders inside must be writable:

- `\cache`
- `\settings`
- `\content`
- `\media`

To make the folders and files writable, use your ftp programm, click on the folder, choose `permissions` and change the permission to `744`. Use the recursive permission for all containing files and folders. If `744` does not work, try `774`.

## htaccess 

If you run your website with https (recommended) or if you want to redirect www-urls to non-www urls, then please check the htaccess file in the root folder. There are several use cases already prepared and you can simply uncomment them, if needed. 

## Run Locally

If you are a developer and if you want to run TYPEMILL locally, then simply download TYPEMILL (zip or git) and visit your local folder like `localhost/typemill`. No additional work is required.

## Error Reporting

TYPEMILL ships with error reporting off, because errors should never been displayed on a live system. But if you run TYPEMILL locally, you can switch error reporting on. Simply open the file `settings.yaml` in the `settings` folder and add a new line:

````
displayErrorDetails: true
````

Don`t forget to turn error reporting off again before you deploy your website live.

