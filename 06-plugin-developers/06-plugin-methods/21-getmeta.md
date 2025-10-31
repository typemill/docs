# getMeta

The method `$this->getMeta()` retrieves all information about html-meta-tags for the frontend.

## Return

* `array` containing meta information.

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
        $metatags = $this->getMeta();
    }
}
```

