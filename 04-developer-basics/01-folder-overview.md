# Folder Overview

The Typemill application has the following folder structure:

* /**cache**: Stores cached files like 
  * Generated CSS files for themes.
  * The Twig template cache if activated.
  * The sitemap.xml
* /**content**: Contains all website content, stored as plain files and folders 
  * Markdown files (`.md`) for page content. 
  * YAML files (`.yaml`). Typemill does not use frontmatter in markdown files. Instead it stores all configurations separately in yaml files with the same name like the markdown files. 
* /**data**: Stores all kinds of additional data like
  * Cached navigation.
  * Data created and used by plugins.
  * Temporary or internal data.
* /**media**: Stores all uploaded media files, sorted into subfolders.
  * **original**: The original uploaded image files.
  * **live**: Resized images used on the live website.
  * **thumb**: Thumbnails shown in the media library.
  * **custom**: Custom image sizes defined and used in themes.
  * **tmp**: Temporary storage for images during upload and before processing.
them. 
  * **files**: All non-image files (like PDFs, docs, zips, movies, audio).
* /**settings**: Stores all settings and user data.
  * **users**: User accounts stored as YAML files.
  * **settings.yaml**: Main system settings including themes and plugins.
  * **secrets.yaml**: All secrets of the type password stored in a separate file that is only loaded if requested explicitly.
  * **kixotesettings.yaml**: Settings for Kixote AI features (like custom prompts).
  * **publickey.pem**: Public key for validating the license or making calls to remote services (e.g. check for updates).
* /**system**: Core code of Typemill
  * /**vendor**: All third-party-libraries used by Typemill (like slim framework and others).
  * /**typemill**: Core classes, services, and backend logic of the Typemill application.
* /**plugins**: Contains all installed plugins.
* /**themes**: Contains all installed themes.

