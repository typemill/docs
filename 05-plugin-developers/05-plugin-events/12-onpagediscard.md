# onPageDiscard

This event is triggered after a changes for a page have been discarded in the author interface. This can be useful if you want to sync data related with the page.

## Availability

This event is only available for the API endpoint (delete) /api/v2/article/discard.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | mixed | - | Array with new markdown, username and item. | 

## Example Data

```
array
(
    'newMarkdown'       => (string)$markdown,
    'username'          => (string)$username,
    'item'              => (object)$item,
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
            'onPageDiscard' => 'onPageDiscard'
        ];
    }

    public function ononPageDiscard($plugindata)
    {
        $pagedata = $plugindata->getData();

        // do something and return the page data
    }
}
```

## See also

* [Item variable](/theme-developers/theme-variables/item) for theme developers.

