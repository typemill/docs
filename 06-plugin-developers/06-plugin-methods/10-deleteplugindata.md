#  deletePluginData

The method `$this->deletePluginData($filename)` deletes a file associated with the current plugin.

## Parameter

| Parameter | Type   | Required | Description                                   |
|-----------|--------|----------|-----------------------------------------------|
| $filename | string | Yes      | The name of the file to delete.               |

## Return 

* `true` if the data is successfully stored. 
* `string` error message if an error occurs during the storage process.

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
        $this->deletePluginData('mydata.yaml');
    }
}
```

