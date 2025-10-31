#  getPluginSettings

The method `$this->getPluginSettings($pluginname = false)` retrieves the settings of a specified plugin from the Typemill settings.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| `$pluginname`| string/bool       | No       | The name of the plugin to retrieve settings for. Defaults to false, which retrieves settings for the currently active plugin. |

## Return 

* `array|false` Returns an array containing the settings of the specified plugin if found, otherwise returns false.

## Example Usage

```php
<?php

namespace Plugins\Myplugin;

use \Typemill\Plugin;

class Myplugin extends Plugin
{
    ...

    public function myFunction()
    {
        $pluginSettings = $this->getPluginSettings();

        # Alternative way: 
        $settings = $this->getSettings();
        $pluginSettings = $settings['plugins']['myplugins'];
    }
}
```

