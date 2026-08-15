# Upgrade Plugins

The upgrade guide for plugins is not ready yet. Please be patient.

## getPath

Instead of `getPath` use:

* `$this->route`
* `$this->adminroute`
* `$this->editorroute`

## getPluginSettings

You can use `getPluginSettings()` without passing the plugin-name as parameter. Typemill will find the name automatically now.

## handle plugin data

Each plugin can write and read its own data now very easily. Just use these methods: 

* `$this->storePluginData($filename, $data)`
* `$this->storePluginYamlData($filename, array $yamlData)`
* `$this->getPluginData($filename)`
* `$this->getPluginYamlData($filename)`

Typemill will automatically use the "/data/"- folder and create a new folder with the name of your plugin. Inside that folder, all data from your plugin will be stored. 

