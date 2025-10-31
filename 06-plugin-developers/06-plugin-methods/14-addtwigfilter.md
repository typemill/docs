#  addTwigFilter

The method `$this->addTwigFilter($name, $filter)` adds a filter to Twig.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $name      | string              | Yes      | The name of the filter for the template.                                                                     |
| $filter    | \Twig\TwigFilter    | Yes      | The filter-function.                                                                          |

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
        $this->addTwigFilter('rot13', function($string){
            return str_rot13($string);
        });
    }
}
```

Use it in your template like this:

````
{{ rot13('this is a text') }}
````

Read the [Twig-Documentation](https://twig.symfony.com/doc/3.x/) for more details.

