# onCspLoaded

This event is triggered right after all plugins and themes got loaded. This event is important if your plugin integrates ressources or content from external websites. In this case you have to add the external domains to the CSP rules, otherwise the content will not load. 

## Availability

This event is always available.

## Data

Plugins **must** return an array for this event; otherwise, domains from other plugins will not be whitelisted in the frontend.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Initially an empty array or an array of domains. | 
| setData | array | required | The initial array with or without modifications. |

## Example Data

```
Array
(
   [0] => 'cdn.jsdelivr.net'
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
            'onCspLoaded' => 'onCspLoaded'
        ];
    }

    public function onCspLoaded($csp)
    {
        $data = $csp->getData();

        $pluginSettings = $this->getPluginSettings();

        if(isset($pluginSettings['tool']) && $pluginSettings['tool'] == 'mathjax')
        {
            $data[] = 'cdn.jsdelivr.net';
        }

        $csp->setData($data);
    }
}
```

## See also

* [About CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) on Mozilla Developer Network.
* [Developer tab](/admin-guide/developer-tab) in the admin area with csp features.
* [Security information](/admin-guide/security) for admins.

