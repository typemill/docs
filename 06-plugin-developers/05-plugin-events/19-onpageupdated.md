# onPageUpdated

This event is triggered after a page has been deleted in the author interface. The event can be useful if you want to sync additional data.

## Availability

This event is only available for the API endpoint (put) /api/v2/draft.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | mixed | - | Returns an array with the old markdown, the new markdown, the username, the item. | 

## Example Data

```
array(
    'oldMarkdown'       => (string),
    'newMarkdown'       => (string),
    'username'          => (string),
    'item'              => (object),
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
            'onPageUpdated' => 'onPageUpdated'
        ];
    }

    public function onPageUpdated($data)
    {
        $pageData       = $data->getData();
        $item           = $pageData['item'];
        $oldMarkdown    = $pageData['oldMarkdown'];
        $newMarkdown    = $pageData['newMarkdown'];
        $username       = $pageData['username'];

        # do something 
    }
}
```

