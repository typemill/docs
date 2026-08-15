#  addTwigFunction

The method `$this->addTwigFunction($name, $function)` adds a function to Twig.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $name      | string              | Yes      | The name of the function.                                                                   |
| $function  | \Twig\TwigFunction  | Yes      | The function to add.                                                                        |

## Example Usage

```php
<?php

namespace Plugins\Myplugin;

use \Typemill\Plugin;

class Myplugin extends Plugin
{
    ...

    public function myFunction()
    {
       $this->addTwigFunction('myName', function(){
          return 'My name is ';
       });
    }
}
```

Use it in your twig-template like this:

````
{{ myName() }}
````

Read the [Twig-Documentation](https://twig.symfony.com/doc/3.x/) for more details.

