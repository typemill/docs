# onPluginsLoaded

This event is triggered after all active plugins have been loaded. The event is useful if you want to check if other plugins are active.

## Availability

This event is everywhere available.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Array with plugin names and class names for each plugin. | 

## Example Data

```
Array
(
    [0] => Array
        (
            [className] => \Plugins\contactform\contactform
            [name] => contactform
        )

    [1] => Array
        (
            [className] => \Plugins\doc\doc
            [name] => doc
        )

    [2] => Array
        (
            [className] => \Plugins\html\html
            [name] => html
        )

    [3] => Array
        (
            [className] => \Plugins\mermaid\mermaid
            [name] => mermaid
        )

    [4] => Array
        (
            [className] => \Plugins\revisions\revisions
            [name] => revisions
        )

    [5] => Array
        (
            [className] => \Plugins\search\search
            [name] => search
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
            'onContentArrayLoaded' => 'onContentArrayLoaded'
        ];
    }

    public function onContentArrayLoaded($plugindata)
    {
        $contentArray = $plugindata->getData();

        // do something like checking if another plugin is available
    }
}
```

