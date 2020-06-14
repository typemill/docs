# Update

Typemill automatically checks the version of your system, of your themes, and of your plugins. If anything is not up to date anymore, then Typemill will add an update banner notification in the settings area. 

## Minor Update

To update Typemill, simply download the latest version of Typemill from the [Typemill website](https://typemill.net) or clone the latest [GitHub code](https://github.com/trendschau/typemill). Then delete the old `system` folder on your server and upload the new system folder. You don't have to touch any other folders.

After you updated your installation, please login to your website and check out the new features.

## Major Update

Sometimes there are a lot of changes that require a deeper update. You can update the whole system like this:

* Backup your settings folder
* Backup your content and media folders
* Delete all other folders:
  * cache
  * plugins
  * system
  * themes
* Upload the new folders

In very rare cases it might be helpful to create a fresh installation. To do so, just delete the settings folder. Make sure that you have created a backup of your old settings before you delete the live settings. After that you can go to `yourwebsite.com/tm/setup`, create a new user, and setup your whole system again. This includes configuring of all your themes and plugins.

## GitHub and Composer

If you work with GitHub and Composer, then make sure that you **always** run the `composer update` command after you upload the new system-folder from GitHub. This is essential, because the GitHub-folder does NOT include the vendor folder with all the dependencies that Typemill uses. If you don't update these dependencies with Composer, then the system will not run. 

If you download the Typemill CMS from https://typemill.net, then you don't have to worry about this, because the vendor folder with all dependencies is included there.

We decided to skip the vendor folder on GitHub because it constantly caused errors due to some missing libraries.

