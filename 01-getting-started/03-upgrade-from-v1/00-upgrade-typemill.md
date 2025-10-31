# User Guide: Upgrade Typemill

Version 2 is out and it is time to update your old version 1 websites. Be aware that old plugins and themes are not compatible with Typemill version 2. Please wait for the updated [plugins](https://plugins.typemill.net) and [themes](https://themes.typemill.net) starting with the version number 2.0.0.

For an update you need a **fresh** installation of Typemill version 2. Do not try to use your old installation, it won't work!!

## 5 Steps to Upgrade

However, an upgrade is as simple as that:

* Create a backup of your Typemill Version 1 website.
* Make a **separate** fresh installation of Typemill Version 2 (locally).
* Copy your content (text, images, files) from your old to your new installation.
* Reconfigure your settings (system, plugins, and themes).
* Upload your new website again.

! Remember that you live in a flat-file world. You can **do the whole upgrade locally (localhost) and upload your website to your live server when everything works fine**!! If you test your new Typemill 2 website on localhost first and then move it to the live environment, remember to delete the cached navigation (folder -> data -> navigation) so Typemill can regenerate the navigation with the new live-urls.

## Step 1: Create a Backup

- Download all your website files to your local machine to create a backup.
- You will need this backup to restore your content and data after a fresh installation.

## Step 2: Perform a Fresh Installation

* Ensure that your server is running **PHP 8.0.0** as this version is supported by both Version 1 and Version 2 of Typemill.
* Download Typemill Version 2 from the [startpage](https://typemill.net).
* Create a fresh setup of the version 2 website with a new admin user.
* We recommend to do this on **localhost** first, but you can also do it on the live-server directly.

Performing a fresh installation is necessary to ensure that all the new files for Version 2 of Typemill are properly set up. You can refer to the [installation](/getting-started/installation) guide for assistance.

## Step 3: Restore Your Content

Copy and paste the following files and folders from your old typemill installation (version 1) to your new typemill installation (version 2):

* `/content`: Copy the whole folder.
* `/media`: Copy the whole folder.
* `/data`: Copy the whole folder.
* `/settings/users`: Copy all users.
* `/cache/custom-themename.css`: If you have individual styles for your theme, then copy the css-file into the cache-folder of version 2. All other cache-files are not needed anymore.
* `.htaccess`: Do NOT copy the old htaccess-file, it will not work with version 2. If you had any individual settings in your htaccess, then copy them manually into the new htaccess-file.

## Step 4: Manual Configuration

You will need to configure your system and themes manually once again. This ensures that no old configurations will interfere with your new website. Alternatively, if you find this process time-consuming, you can copy and paste selected configurations from your old settings.yaml file into your new settings.yaml file. However, this is not recommended because it might lead to errors if done wrong.

## Step 5: Upload Your New Website

If you finished and tested your update on localhost first (recommended), then you can simply upload your whole website to your live-server with ftp. Do not overwrite the version 1 files with version 2 files. Instead delete version 1 from your server completely and then uload the fresh version 2 to keep everything clean.

That's it!

