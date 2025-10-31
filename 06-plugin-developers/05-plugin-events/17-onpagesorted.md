# onPageSorted

This event is triggered after a page has been sorted with the interactive navigation in the author interface. This can be useful if you want to sync additional data.

## Availability

This event is only available for the API endpoint (post) /api/v2/article/sort.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | The event sends the old item and the new item. | 

## Example Data

```
array(
    'olditem'   => (object)$item,
    'newitem'   => (object)$newitem
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
            'onPageSorted' => 'onPageSorted'
        ];
    }

    public function onPageSorted($data)
    {
        $pagedata = $data->getData();  // Assume $pagedata is an object

        $olditem  = $pagedata['olditem'] ?? false;
        $newitem  = $pagedata['newitem'] ?? false;

        # do something like compare or resort related files 
    }
}
```

## See also

* [Item variable](/theme-developers/theme-variables/breadcrumb) for theme developers.

