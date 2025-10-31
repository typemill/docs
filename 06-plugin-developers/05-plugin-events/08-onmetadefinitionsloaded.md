# onMetaDefinitionsLoaded

This event is triggered after the field-definitions for the meta-data have been loaded. If you want to get the meta-data instead, use [onMetaLoaded](/plugin-developers/plugin-events/onmetaloaded).

## Availability

This event is available for the editor view in the author interface.

## Data

Plugins **must** return an array with meta-definitions for this event; otherwise, the meta-tabs will not be available in the editor.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Multi-dimensional array of meta-definitions. | 
| setData | array | required | Initial or manipulated array with meta-definitions. |

## Example Data

```
Array
(
    [meta] => Array
        (
            [fields] => Array
                (
                    [navtitle] => Array
                        (
                            [type] => text
                            [label] => Navigation Title
                            [maxlength] => 60
                        )

                    [fieldsetcontent] => Array
                        (
                            [type] => fieldset
                            [legend] => Meta-Content
                            [fields] => Array
                                (
                                    [title] => Array
                                        (
                                            [type] => text
                                            [label] => Meta title
                                            [maxlength] => 100
                                        )

                                    [description] => Array
                                        (
                                            [type] => textarea
                                            [label] => Meta description
                                            [size] => 160
                                            [description] => If not filled, the description is extracted from content.
                                        )

                                    [heroimage] => Array
                                        (
                                            [type] => image
                                            [label] => Hero Image
                                            [description] => Maximum size for an image is 5 MB. Hero images are not supported by all themes.
                                        )

                                    [heroimagealt] => Array
                                        (
                                            [type] => text
                                            [label] => Alternative Text for the hero image
                                        )

                                )

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
            'onMetaDefinitionsLoaded' => 'onMetaDefinitionsLoaded'
        ];
    }

    public function onMetaDefinitionsLoaded($plugindata)
    {
        $metadefinitions = $plugindata->getData();

        // do something and return the metadefinitions

        $plugindata->setData($metadefinitions);
    }
}
```

## See also

* [onMetaLoaded](/plugin-developers/plugin-events/onmetaloaded) event for plugin developers with the meta-data.
* [Edit meta information](/author-guide/edit-meta-information) for authors.

