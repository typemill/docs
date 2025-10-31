# getTwig

The method `$this->getTwig()` retrieves the Twig instance. This is helpful when you want to load and render a template in your plugin.

* **@return:** (\Twig\Environment) Returns the Twig instance.

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
        $twig               = $this->getTwig();  // get the twig-object
        $loader             = $twig->getLoader();  // get the twig-template-loader  
        $loader->addPath(__DIR__ . '/templates');

        // now render the template with some variables in it.
        $twig->fetch('/yourTemplate.twig', array('mykey' => 'myvalue'));
    }
}
```

