# onShortcodeFound

This event is triggered when a shortcode is found while the markdown is parsed into html. You can use this event to create new shortcodes and to add shortcode-features to markdown.

## Availability

This event is always triggered if Markdown is parsed into html (frontend-pages, editor-pages in the author interface, or the markdown-function for twig).

## Data

Plugins **must** return data from the shortcode for this event; otherwise, the shortcode will not be available in the frontend.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Array with the name and the params of the shortcode. | 
| setData | mixed | required | Can return a string, an array or something similar that replaces the shortcode. |

## Example Data

```
Array
(
    [name] => div
    [params] => Array
        (
            [class] => test
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
      return array(
        'onShortcodeFound' => 'onShortcodeFound'
      );
    }

    public function onShortcodeFound($shortcode)
    {
      # read the data of the shortcode
      $shortcodeArray = $shortcode->getData();

      # check if it is the shortcode name that we where looking for
      if(is_array($shortcodeArray) && $shortcodeArray['name'] == 'replace')
      {
        # we found our shortcode, so stop firing the event to other plugins
        $shortcode->stopPropagation();

        # and return a html-snippet that replaces the shortcode on the page.
        $shortcode->setData('<span class="tm-green">A replacement from the plugin</span>');
      }
}
```

## See also

* [Shortcode Example](/plugin-developers/examples/shortcodes) for plugin developers.

