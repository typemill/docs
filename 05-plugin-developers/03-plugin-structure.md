# Plugin Structure

To ensure your plugin functions seamlessly, it requires a specific file structure. Only two files are required to start.

## Minimal Plugin Structure

A straightforward plugin only needs one folder and two files: a PHP file containing the plugin code and a YAML file for configurations.

```plaintext
/myPlugin // folder
  - myPlugin.php // file with plugin code
  - myPlugin.yaml // file with YAML configuration
```


## Naming Rules

Use a unique, short, and descriptive name for your plugin with alpha characters [a-z].

* Use the plugin name for your folder name.
* Use the plugin name for the php file.
* Use the plugin name for the YAML configuration file.

## Extended Plugin Structure

If you create a complex plugin with many functionalities, then you should structure your code a bit more. It depends highly on the features that the plugin provides. An example: 

```plaintext
/myPlugin
  - myPlugin.php // basic php logic and entry point for the plugin
  - myPlugin.yaml // yaml configuration
  - myPluginMiddleware.php // Middleware that has been added in the myPlugin.php file. 
  - myPluginRoute.php // Entry point for a new route, that has been added in the myPlugin.php file. 
  - /templates // a folder with twig-templates used by the plugin
  - /js // a folder with javascript-files, for example a library or a vue component for the author area.
  - /vendor // vendor folder with libraries used by the plugin
```

