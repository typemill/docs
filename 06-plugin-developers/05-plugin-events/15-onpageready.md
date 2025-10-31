#  onPageReady

This event is triggered when a page is fully prepared for rendering. The event is useful if you want to add additional data to the frontend before it is displayed to users.

## Availability

This event is available for all frontend pages and for the editor pages in the author environment.

## Data

Plugins can return additional page data as an array. The data will be merged with the existing page data and then be available in the frontend.

| Getter/Setter | Type         | Required | Description                                                            | 
|:--------------|:-------------|:--------|:----------------------------------------------------------------------|
| getData       | empty array  | -       | Retrieves an empty array.                                             | 
| setData       | array        | optional | Array of additional data that will be merged with the existing page data. |

## Example Data

```php
Array
(
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
            'onPageReady' => 'onPageReady'
        ];
    }

    # add a search form to frontend
    public function onPageReady($page)
    {
        $pageData           = $page->getData($page);

        $placeholder = 'Search...'; // Define a placeholder for the search input
        $pageData['widgets']['search'] = '<div class="searchContainer" id="searchForm">' .
                                            '<input type="text" placeholder="' . $placeholder . '" />'.
                                        '</div>';
        $page->setData($pageData);
    }
}
```

