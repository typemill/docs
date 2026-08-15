# onItemLoaded

This event is triggered after the item of the page has been extracted from the navigation. 

## Availability

This event is available for all frontend pages and for the editor pages in the author environment.

## Data

Plugins **must** return the item object for this event; A missing item will lead to unexpected errors in frontend and backoffice.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | object | - | The item object of the page. | 
| setData | object | required | Initial or manipulated item object of the page. |

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
            'onItemLoaded' => 'onItemLoaded'
        ];
    }

    public function onItemLoaded($itemService)
    {
        $item = $itemService->getData();
        if ($item->elementType == 'folder') {
            $this->addMeta('rss', '<link rel="alternate" type="application/rss+xml" title="' . $item->name . '" href="' . $item->urlAbs . '/rss">');
        }
    }
}
```

## See also

* [?](/theme-developers/theme-variables/item) for theme developers.

