#  addJS

The method `$this->addJS($JS)` adds a reference to a JavaScript file to be included in the page.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $JS       | string              | Yes      | The relative path to the JavaScript file.                                                 |

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
       $this->addJS('https://some-url.com/with/path/to/javascript.js');

       $this->addJS('/yourpluginfolder/subfolder/javascript.js');
    }
}
```

Will be rendered in the template with this template tag:

```
{{ renderJS() }} // This tag should be placed before the closing body tag.
```

