#  addCSS

The method `$this->addCSS($CSS)` adds a reference to a CSS file to be included in the page.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $CSS      | string              | Yes      | The relative path to the CSS file.                                                         |

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
        $this->addCSS('https://some-url.com/with/path/to/style.css');

        $this->addCSS('/yourpluginfolder/subfolder/style.css');
    }
}
```

Will be rendered in the template with this template tag:

```
{{ renderCSS() }} // This tag should be placed in the HTML header.
```

