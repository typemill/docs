#  onPageRenamed

This event is triggered after the slug of a page has been changed in the author interface. This can be useful if you want to sync additional data.

## Availability

This event is only available for the API endpoint (post) /api/v2/article/rename.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type  | Required | Description                                   | 
|:--------------|:-----|:--------|:----------------------------------------------|
| getData       | array | -       | Array with the old item and the new relative URL. | 

## Example Data

```
array(
    'item'      => (object)$item,
    'newUrl'    => (string)$newUrl
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
            'onPageRenamed' => 'onPageRenamed'
        ];
    }

    public function onPageRenamed($data)
    {
        $this->settings     = $this->getSettings();
        $this->storagePath  = $this->settings['rootPath'] . DIRECTORY_SEPARATOR . 'data' . DIRECTORY_SEPARATOR . 'revisions';

        $pagedata   = $data->getData();

        $olditem       = $pagedata['item'] ?? false;
        $newurl        = $pagedata['newUrl'] ?? false;

        if (!$olditem && !$newurl)
        {
            return false;
        }

        $oldfilename    = $this->getFileName($olditem);
        $newfilename    = $this->getFilenameFromUrl($newurl);

        $oldfiles       = glob($this->storagePath . DIRECTORY_SEPARATOR . $oldfilename . '*');

        foreach ($oldfiles as $oldfile)
        {
            $newfile = str_replace($oldfilename, $newfilename, $oldfile);

            if (file_exists($oldfile))
            {
                rename($oldfile, $newfile);
            }
        }
    }
}
```

## See also

* [item variable](/theme-developers/theme-variables/item) for theme developers.

