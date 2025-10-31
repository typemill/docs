# onPagePublished

This event is triggered after a page has been published in the author interface. The event sends all available data for the page. This can be useful if you want to sync or send the data.

## Availability

This event is only available for the API endpoint (put) /api/v2/article.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | mixed | - | An array with the item, the username, the markdown, and the metadata. | 

## Example Data

```
array(
  'markdown' => (string)
  'username' => (string)
  'item' => (object)
  'metadata' => (array)
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
            'onPagePublished' => 'onPagePublished'
        ];
    }

    public function onPagePublished($data)
    {
        $pageData       = $data->getData();
        $item           = $pageData['item'] ?? false; 
        $markdown       = $pageData['markdown'] ?? false;
        $username       = $pageData['username'] ?? false;

        # do something
    }
}
```

## See also

* [Item variable](/theme-developers/theme-variables/item) for theme developers.
* [Metadata variable](/theme-developers/theme-variables/metadata) for theme developers.

