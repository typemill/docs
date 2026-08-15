# onPageUnpublished

This event is triggered after a page has been deleted in the author interface. The event can be useful if you want to sync additional data.

## Availability

This event is only available for the API endpoint (delete) /api/v2/article.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | object | - | Sends the item of the deleted page. | 

## Example Data

```
stdClass Object
(
    [status] => published
    [originalName] => home
    [elementType] => folder
    [fileType] => md
    [order] => 
    [name] => home
    [slug] => 
    [path] => 
    [pathWithoutType] => /index
    [key] => 
    [keyPath] => 
    [keyPathArray] => Array
        (
            [0] => 
        )

    [chapter] => 
    [urlRel] => /
    [urlRelWoF] => /
    [urlAbs] => http://localhost/typemill
    [active] => 1
    [activeParent] => 
    [hide] => 
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
            'onPageUnpublished' => 'onPageUnpublished'
        ];
    }

    public function onPageUnpublished($item)
    {
        $this->updateRssXmls();
    }
}
```

## See also

* [Item variable](/theme-developers/theme-variables/breadcrumb) for theme developers.

