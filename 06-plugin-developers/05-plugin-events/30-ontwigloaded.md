# onTwigLoaded

This event is triggered when the twig extension has been loaded. Use it whenever you want to add something like templates, JavaScript, Vue applications, or CSS. 

## Availability

This event is available for all frontend-websites, author and admin-interfaces, and api endpoints.

## Data

This method does not send any data. 

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
            'onTwigLoaded' => 'onTwigLoaded'
        ];
    }

    public function onTwigLoaded()
    {
        # add only to frontend pages
        if(!$this->adminroute)
        {
            $pluginSettings = $this->getPluginSettings();
            if(isset($pluginSettings['buttonstyle']) && $pluginSettings['buttonstyle'])
            {
                $this->addCSS('/clipboard/public/clipboard.css');
            }

            $this->addJS('/clipboard/public/clipboard.min.js');
            $this->addJS('/clipboard/public/initClipboard.js');
        }
    }
}
```

