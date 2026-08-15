# onSessionSegmentsLoaded

This event is triggered after the url segments for starting a session have been loaded. The event is useful if you want to start a session for a specific page or integration. 

## Availability

This event is everywhere available.

## Data

Plugins **must** return the array with session segments for this event; otherwise, sessions for other plugins will not be started. Segments from plugins will be merged with default segments later.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Empty array or one dimensional array with url segments from other plugins. | 
| setData | array | required | Original or manipulated array with session segments. |

## Example Data

```
Array
(
    [0] => sort
    [1] => /sort
    [2] => contactprocessor
    [3] => /contactprocessor
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
            'onSessionSegmentsLoaded' => 'onSessionSegmentsLoaded'
        ];
    }

    # add the path stored in user-settings to initiate session
    public function onSessionSegmentsLoaded($segments)
    {
        $this->settings         = $this->getSettings();
        $this->pluginSettings   = $this->getPluginSettings();
        $this->contactpage      = trim($this->pluginSettings['page_value'], '/');

        # get url-segments with cookies on
        $data   = $segments->getData();

        # add the page for contact form to the segments with cookies
        $data[] = $this->contactpage;
        $data[] = '/' . $this->contactpage;
        $data[] = 'contactprocessor';
        $data[] = '/contactprocessor';

        $segments->setData($data);
    }
}
```

