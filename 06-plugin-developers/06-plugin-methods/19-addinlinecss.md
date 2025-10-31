#  addInlineCSS

The method `$this->addInlineCSS($CSS)` adds inline CSS code to be included in the page.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $CSS       | string              | Yes      | The inline CSS code to add.                                                                 |

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
        $this->addInlineCSS('body{ background: #000; }');
    }
}
```

Will be rendered in the template with this template tag:

```
{{ renderCSS() }} // This tag should be placed in the HTML header.
```

