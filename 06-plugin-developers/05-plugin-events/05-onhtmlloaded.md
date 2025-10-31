# onHtmlLoaded

This event is triggered after the markdown content of a page has been converted into html by the parsedown library.

## Availability

This event is available for all frontend pages.

## Data

Plugins **must** return a html-string for this event; otherwise, the content will not be available in the frontend. Please note that the HTML-string will **not contain** the first h1 element.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | string | - | HTML formatted string of the page content. | 
| setData | string | required | Initial or manipulated string with HTML-format. |

## Example Data

```
<p>Typemill is a lightweight, flat-file CMS designed for simple, fast, and flexible website and eBook creation using Markdown. Create handbooks, documentations, manuals, reports, traditional websites, online novels, and more.</p>
<p>Stay in the loop and subscribe to the <a href="https://typemill.net/news">Typemill newsletter</a>!</p>
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
            'onHtmlLoaded' => 'onHtmlLoaded'
        ];
    }

    public function onHtmlLoaded($html)
    {

        if($this->route == $this->contactpage)
        {
            $content = $html->getData();

            # get the public forms for the plugin
            $contactform = $this->generateForm('contact.send');

            # add forms to the content
            $content = $content . '<div class="tm-contactform">' . $contactform . '</div>';

            $html->setData($content);
        }
    }
}
```

## See also

* [onMardkownLoaded](/plugin-developers/plugin-events/onmarkdownloaded) event for plugin developers.
* [onContentArrayLoaded](/plugin-developers/plugin-events/oncontentarrayloaded) event for plugin developers.

