# onPageCreated

This event is triggered after a page has been created in the author interface. This can be useful if you want to sync data related with the page.

## Availability

This event is only available for the API endpoint (post) /api/v2/article.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | mixed | - | Array with markdown, metadata, itempath and username. | 

## Example Data

```
array
(
    'markdown'  => (string)$markdown, 
    'metadata'  => (array)$metadata,
    'itempath'  => (string)$itempath,
    'username'  => (string)$username
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
            'onPageCreated' => 'onPageCreated'
        ];
    }

    public function onPageCreated($plugindata)
    {
        $pagedata = $plugindata->getData();
    }
}
```

