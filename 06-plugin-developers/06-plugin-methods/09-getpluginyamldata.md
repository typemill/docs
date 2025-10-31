#  getPluginYamlData

The method `$this->getPluginYamlData($filename)` retrieves YAML formatted data from a file associated with the current plugin.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| `$filename`| string              | Yes      | The name of the file to retrieve data from.                                               |

## Return 

* `array|false` Returns an array containing the YAML data from the specified file if found, otherwise returns false.

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
        $this->getPluginYamlData('mydata.yaml');
    }
}
```

