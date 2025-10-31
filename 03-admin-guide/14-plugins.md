#  Plugins

Typemill provides various plugins and customization options in the plugin section of the system area. Here, you can add, activate, and configure the plugins.

![Screenshot of plugin configuration in Typemill](media/live/typemill-plugins.webp){.center loading="lazy" width="820" height="533"}

## Plugin Directory

You can select and download plugins for Typemill from the [plugin directory](https://plugins.typemill.net). Many plugins are free to use. Plugins that require a license include a button indicating the specific license: MAKER or BUSINESS.

## Install a Plugin

You can add as many plugins to your Typemill website as you want. Just download a new plugin from the plugin directory, unzip it, and then upload it (e.g., with FTP) to the plugin folder of your Typemill website (e.g., `/plugin/mermaid`).

After uploading the plugin, the folder structure should look like this:

```YAML
/typemill
  /plugins
    /mermaid
     - mermaid.yaml
     - mermaid.php
     - [more plugin files and folders]
```

After uploading the plugin to your Typemill installation, please refresh the page with the listed plugins, and the new plugin should be part of the list. To activate a plugin, just click on the activate checkbox at the top.

## Configure a Plugin

Each plugin can have very individual configuration options. To open the configuration options, click on the configuration button of a plugin. Some plugins add new items to the system navigation, where you have more configuration options. The plugin directory provides more details for each plugin.

![Configuration option for a Typemill plugin](media/live/plugin-configuration.webp){.center loading="lazy" width="820" height="529"}

## Update a Plugin

If there is a new version of a plugin available, you will see a small update banner in the plugin area of the author panel. To update a plugin, simply open your FTP software, go to the plugin folder of Typemill, delete the plugin, and upload the new version of the plugin. Then check the plugin configuration in the settings of the author panel.

## Use Bundles

The Typemill website also provides bundles for specific use cases that come with a selection of helpful plugins and themes for different scenarios. Check out the [different solutions](https://typemill.net/solutions) on the Typemill website.

## Develop a Plugin

If you are a developer and want to create your own plugins, please refer to the [plugin documentation](/plugin-developers).

