# settings

The Twig-variable `{{ settings }}` contains all settings of Typemill as an array, such as:

* The user settings for the system.
* The user settings for the themes.
* The user settings for the plugins.

## System Settings

The following settings might be useful for your theme:

### {{ settings.title }}

The title of the website from the system settings. The default value is `TYPEMILL`.

### {{ settings.author }}

The author of the website from the system settings. The default value is `unknown`.

### {{ settings.langattr }}

the language attribute of the website from the system settings. You should default this to `en` in your theme like this:

```Twig
{{ settings.langattr | default('en') }}
```




### {{ settings.copyright }}

The Copyright information of the website. The default value is `copyright`.

### {{ settings.theme }}

The name of the theme that is in use. Default value is `typemill`.

### {{ settings.version }}

The version of TYPEMILL that is in use. A value of the format `0.0.1`.

## Settings for Themes and Plugins

You have also access to all plugin and theme settings. Simply use them like this: 

````
{{ settings.themes.mytheme.mythemesetting }}
{{ settings.plugins.myplugin.mypluginsetting}}
````






You have to replace `mytheme` and `myplugin` with the name of the theme or plugin, and the `mythemesettings` and `mypluginsettings` with the name of the settings you use. You can find the name of the settings in the YAML-file of the plugin or theme. 

For example: The standard theme of TYPEMILL is called `typemill`. And there you can edit the label of the start button displayed on the startpage. The name of the variable is `start`. So you can simply print it out like this: 

````
<button>{{ settings.themes.typemill.start }}</button>
````

