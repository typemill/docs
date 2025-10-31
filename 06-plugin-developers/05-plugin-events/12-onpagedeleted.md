# onPageDeleted

This event is triggered after a page has been deleted in the author interface. The event sends the item of the deleted element. This can be useful if you want to sync data related with the page.

## Availability

This event is only available for the API endpoint (delete) /api/v2/article.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | object | - | The item object of the deleted page. | 

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
            'onPageDeleted' => 'onPageDeleted'
        ];
    }

    public function onPageDeleted($data)
    {
        # delete a revision-file, an archived file, or something similar
        $this->settings     = $this->getSettings();
        $item = $data->getData();

        $this->storagePath  = $this->settings['rootPath'] . DIRECTORY_SEPARATOR . 'data' . DIRECTORY_SEPARATOR . 'revisions';

        $item = $data->getData();

        $filename       = $this->getFileName($item);
        $filepath       = $this->storagePath . DIRECTORY_SEPARATOR . $filename . '.json';

        if(file_exists($filepath))
        {
            unlink($filepath);
        }
    }
}
```

## See also

* [Item variable](/theme-developers/theme-variables/item) for theme developers.

