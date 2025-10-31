# onTwigGlobalsLoaded

This event is triggered at the start of the application live cycle. Use it if you want to add a twig global later so that the global can be initialized in slim.

## Availability

This event is everywhere available.

## Data

Plugins **must** return an array of twig globals for this event; otherwise, the globals will not work. 

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Array with global names as key (initially empty). | 
| setData | array | required | Initial or manipulated array of twig globals. |

## Example Data

```
Array
(
    [errors] => (bool)
    [flash] => (bool)
    [assets] => (bool)
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
            'onTwigGlobalsLoaded' => 'onTwigGlobalsLoaded'
        ];
    }

    public function onTwigGlobalsLoaded($data)
    {
        $globals = $data->getData();

        $globals['providerlist'] = true;

        $data->setData($globals);       
    }

    public function myFunction()
    {
       $this->addTwigGlobal('providerlist', new Text());
    }
}
```

## See also

* [addTwigGlobal method](/plugin-developers/plugin-methods/addtwigglobal) for plugin developers.

