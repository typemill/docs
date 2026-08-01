#  Updates

Typemill automatically checks the version of your system, your themes, and your plugins. If anything is not up to date, Typemill will show an update banner.

There is no automatic update feature yet, but a manual update can be quickly done with FTP (or Git and Composer).

## Update Typemill

A simple update of a new Typemill release is done in three steps and should not require more than 10 minutes for one website:

1. Download the latest version of Typemill from the [Typemill website](https://typemill.net) or clone the latest [GitHub code](https://github.com/trendschau/typemill).
2. Delete the content of the folder `system` on your server. This includes:
   - Folder `typemill`: which contains all Typemill code.
   - Folder `vendor`: which includes all 3rd-party libraries.
   - The file `autoload.php`.
3. Upload the new folders `typemill` and `vendor`, along with the new file `autoload.php`, to your server.

Quickly check the new version number in the system settings of your dashboard, and you are done.

!!! **Tip: Avoid Errors in Frontend**
!!! 
!!! Sometimes users overwrite the old files on the server with the new files instead of deleting the old files first. This method avoids downtime but is not recommended, as it can lead to problems and errors if overwriting fails. Instead, delete the files first and then upload the new files.
!!! 
!!! To avoid errors in the frontend during the update, you can delete/backup the `index.php` file in the root folder and upload it again after the upload. Usually, a standard server message will appear, indicating that the website is under construction.

## Update Plugins and Themes

Updating themes and plugins is quickly done with similar steps:

1. Download the new version of a plugin or theme and unzip it.
2. Delete the old plugin or themes from the folders `plugins` or `themes`.
3. Upload the new plugin or theme folder.

You are done!

## Update with Git and Composer

If you work with GitHub and Composer, make sure that you **always** run the `composer update` command. This is essential because the GitHub folder does NOT include the vendor folder with all the dependencies that Typemill uses. If you don't update these dependencies with Composer, the system will not run.

