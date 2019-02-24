# Settings

TYPEMILL provides detailed settings and you have access to nearly all settings in the author panel. You will find everything in the `settings` tab:

* System settings
* Theme settings
* Plugin settings
* User settings

All settings are stored in the `\settings` folder of TYPEMILL. It is not recommended to edit the settings manually, because it might crash the system if done wrong.

## Advanced Settings

There are some settings that are not available via the author panel. Most of them are not really useful, but if you are a developer and if you develop a theme or a plugin locally, you probably want to display a detailed error report. To do so, simply add the following line to the settings.yaml: 

````
displayErrorDetails: true
````

Don't forget to set it back to `false` before you deploy the website live. It is not secure to show the world your internal errors and many hosters will turn off all public error reports by default.

