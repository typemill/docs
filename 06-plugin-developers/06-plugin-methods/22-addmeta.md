#  addMeta

The method `$this->addMeta($key, $meta)` adds html meta-tags for the frontend.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $key       | string              | Yes      | The key of the meta information.                                                            |
| $meta      | mixed               | Yes      | The value of the meta information.                                                          |

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
        $this->addMeta('noindex','<meta name="robots" content="noindex">');
    }
}
```

