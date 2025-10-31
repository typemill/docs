#  addTwigGlobal

The method `$this->addTwigGlobal($name, $class)` adds a global variable to Twig. Note that you have to initialize a global name with [onTwigGlobalsLoaded]( /plugin-developers/plugin-events/ontwigglobalsloaded) before you can add a global.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $name      | string              | Yes      | The name of the global variable.                                                            |
| $class     | mixed               | Yes      | The value of the global variable.                                                           |

## Example Usage

```php
<?php

namespace Plugins\Myplugin;

use \Typemill\Plugin;

class Myplugin extends Plugin
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

        $globals['text'] = true;

        $data->setData($globals);       
    }

    public function myFunction()
    {
       $this->addTwigGlobal('text', new Text());
    }
}
```

This will add the class `Text()` to the variable 'text', which you can use in your twig-templates like this:

```twig
  {{ text }}
```

Read the [Twig-Documentation](https://twig.symfony.com/doc/3.x/) for more details.

