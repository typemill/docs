# onPagetreeLoaded

This event is triggered after the content navigation (content-tree) has been created. The event can be useful if you want to change the navigation in the frontend or in the author interface. 

## Availability

This event is available for frontend pages and for the editor views in the author interface.

## Data

Plugins **must** return the navigation for this event; otherwise, the navigation will not be available in the author interface or in the frontend.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Multi-dimensional array of item objects representing the navigation. | 
| setData | array | required | Original or manipulated navigation. |

## Example Data

```
array(
    [0] => stdClass Object
        (
            [originalName] => 00-guides
            [elementType] => folder
            [contains] => pages
            [status] => published
            [fileType] => md
            [order] => 00
            [name] => Guides
            [slug] => guides
            [path] => /00-guides
            [pathWithoutType] => /00-guides/index
            [urlRelWoF] => /guides
            [urlRel] => /typemill/guides
            [urlAbs] => http://localhost/typemill/guides
            [key] => 0
            [keyPath] => 0
            [keyPathArray] => Array
                (
                    [0] => 0
                )

            [chapter] => 1
            [active] => 1
            [activeParent] => 
            [hide] => 
            [folderContent] => Array
                (
                    [0] => stdClass Object
                        (
                            ...
                        )
                 )
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
            'onPagetreeLoaded' => 'onPagetreeLoaded'
        ];
    }

    public function onContentArrayLoaded($plugindata)
    {
        $navigation = $plugindata->getData();

        // do something and return the navigation

        $plugindata->setData($navigation);
    }
}
```

## See also

* [navigation variable](/theme-developers/theme-variables/navigation) for theme developers.
* [Item variable](/theme-developers/theme-variables/item) for theme developers.

