# onMarkdownLoaded

This event is triggered after the markdown content of a page has been loaded and before it is converted into HTML.

## Availability

This event is available for all frontend pages and for the editor view in the author interface.

## Data

Plugins **must** return a markdown-string for this event; otherwise, the content will not be available in the frontend.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | string | - | Markdown formatted string of the page content. | 
| setData | string | required | Initial or manipulated string with Markdown-format. |

## Example Data

```
# Guides

A Standard Operating Procedure (SOP) is a detailed, written set of instructions that describes the step-by-step process for carrying out routine activities within an organization. **SOPs** are designed to ensure consistency, efficiency, and quality in performing specific tasks or operations.

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
            'onMarkdownLoaded' => 'onMarkdownLoaded'
        ];
    }

    public function onMarkdownLoaded($plugindata)
    {
        $markdown = $plugindata->getData();

        # do something, e.g. add another paragraph

        $plugindata->setData($markdown);
    }
}
```

## See also

* [onHtmlLoaded](/plugin-developers/plugin-events/onhtmlloaded) event for plugin developers.
* [onContentArrayLoaded](/plugin-developers/plugin-events/oncontentarrayloaded) event for plugin developers.

