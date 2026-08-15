# onMetaLoaded

This event is triggered after the meta-data for a page have been loaded. 

## Availability

This event is available for the all frontend-pages.

## Data

Plugins **must** return an array with meta-data for this event; otherwise, the meta-data will not be available in the frontend.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Multi-dimensional array of meta-data. | 
| setData | array | required | Initial or manipulated array with meta-data. |

## Example Data

```
Array
(
    [meta] => Array
        (
            [owner] => trendschau
            [author] => Sebastian Mustermann
            [created] => 2024-08-19
            [time] => 09-04-57
            [navtitle] => Standard Operating Procedure
            [modified] => 2024-08-29
            [title] => Standard Operating Procedure (SOP)
            [description] => 
Target Group: 
Entity: Organisation, Process Management, internal
Software: Process.st, Enterprise Architect

        )

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
            'onMetaLoaded' => 'onMetaLoaded'
        ];
    }

    public function onMetaLoaded($plugindata)
    {
        $metadata = $plugindata->getData();

        // do something and return the metadata

        $plugindata->setData($metadata);
    }
}
```

## See also

* [onMetaDefinitionsLoaded](/plugin-developers/plugin-events/onmetadefinitionsloaded) event for plugin developers with the meta-definitions.
* [Edit meta information](/author-guide/edit-meta-information) for authors.

