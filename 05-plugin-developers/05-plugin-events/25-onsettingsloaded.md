# onSettingsLoaded

This event is triggered after all settings have been merged and loaded. If you want to use settings in your plugin, you should use the methods [getSettings method](/plugin-developers/plugin-methods/getsettings) or [getPluginSettings method](/plugin-developers/plugin-methods/getpluginsettings) instead.

## Availability

This event is everywhere available.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Multi-dimensional array with all settings. | 

## Example Data

```
Array
(
    [version] => 2.8.0
    [title] => Typemill
    [author] => Sebastian Schürmanns
    [copyright] => ©
    [language] => en
    [langattr] => en
    [editor] => visual
    [storage] => \Typemill\Models\Storage
    [formats] => Array
        (
            [0] => markdown
            [1] => headline
            [2] => ulist
            [3] => olist
            [4] => table
            [5] => quote
            [6] => notice
            [7] => image
            [8] => video
            [9] => file
            [10] => toc
            [11] => hr
            [12] => definition
            [13] => code
            [14] => shortcode
        )
    ...
)
```

## Example Usage

```php
<?php

namespace plugins\myplugin;

use \typemill\plugin;

class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'onSettingsLoaded' => 'onSettingsLoaded'
        ];
    }

    public function onSettingsLoaded($plugindata)
    {
        $settings = $plugindata->getData();
    }
}
```

## See also

* [getSettings method](/plugin-developers/plugin-methods/getsettings) for plugin developers.
* [getPluginSettings method](/plugin-developers/plugin-methods/getpluginsettings) for plugin developers.

