#  addInlineJS

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $JS       | string              | Yes      | The inline JavaScript code to add.                                                         |

The method `$this->addInlineJS($JS)` adds inline JavaScript code to be included in the page.

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
        $this->addInlineJS('alert("hello");');
    }
}
```

Will be rendered in the template with this template tag:

```
{{ renderJS() }} // This tag should be placed before the closing body tag.
```

