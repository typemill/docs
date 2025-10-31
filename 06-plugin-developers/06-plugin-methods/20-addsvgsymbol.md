#  addSvgSymbol

The method `$this->addSvgSymbol($symbol)` adds an SVG symbol.

## Parameter

| Parameter  | Type   | Required | Description                              |
|------------|--------|----------|------------------------------------------|
| $symbol    | string | Yes      | The SVG symbol to add.                  |

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
        $this->addSvgSymbol('<symbol id="icon-example" viewBox="0 0 24 24"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2z"/></symbol>');
    }
}
```

