# getSettings

The plugin-method `$this->getSettings()` returns all settings of Typemill as an array.

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
        $settings = $this->getSettings();
        ...
    }
}
```

