# The Structure of a Plugin

Each plugin needs a file structure. The rules for this file structure are easy to learn, and important to follow; otherwise, the plugin won't work.

## Basic Folder And File

All a simple plugin needs to work is one folder and one file. The naming conventions are as follows:

* The name of the folder is the same as the name of the plugin and...
* the name of the core PHP file is the same as the name of the plugin.

So it might look like this:

````
/myPlugin  // folder
  - myPlugin.php  // file with plugin-code
````

## Configuration File

However, if you want to distribute a plugin on the TYPEMILL website (not ready yet), then you **must** add a **configuration file** with YAML syntax, and this file must have the **same name** as the plugin again. This is very important, because the versioning and the update notifications, that are shown in the setup, only work with this configuration file.

So the minimum structure required for a distributed plugin looks like this:

````
/myPlugin // folder
  - myPlugin.php // file with plugin-code
  - myPlugin.yaml // file with yaml-configuration
````

## Other files

You are free to add any other files and folders into your plugin. A plugin can have assets like images, CSS, or JavaScript. It can also have templates, controllers, and middleware. The folder structure of a more complex plugin might look like this:

````
/myPlugin
- myPlugin.php 
- myPlugin.yaml
  /controllers
  - myController.php
  /middleware
  - myMiddleware.php
  /templates
  - mytemplate.twig
  /public
  - mystyle.css
  - myscript.js
  - myimage.jpg
````
