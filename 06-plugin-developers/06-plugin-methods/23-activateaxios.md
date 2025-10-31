# activateAxios

The method `$this->activateAxios()` activates the Axios library for the frontend.

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
        $this->activateAxios();
    }
}
```


Will be rendered in the template with this template tag:

```
{{ renderJS() }} // This tag should be placed before the closing body tag.
```

