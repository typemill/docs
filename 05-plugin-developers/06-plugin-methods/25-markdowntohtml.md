#  markdownToHtml

The method `$this->markdownToHtml($markdown)` converts Markdown to HTML.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $markdown  | string              | Yes      | The Markdown to convert.                                                                     |

## Return 

* `string`: The HTML converted from Markdown.

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
        $html = $this->markdownToHtml('## My Markdown Headline');
    }
}
```

