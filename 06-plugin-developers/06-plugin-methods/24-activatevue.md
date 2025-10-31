# activateVue

The method `$this->activateVue()` activates Vue.js.

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
        $this->activateVue();
    }
}
```


Will be rendered in the template with this template tag:

```
{{ renderJS() }} // This tag should be placed before the closing body tag.
```

