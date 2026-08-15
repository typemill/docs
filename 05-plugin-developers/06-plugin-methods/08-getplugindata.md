#  getPluginData

The method `$this->getPluginData($filename)` retrieves data from a file associated with the plugin. It fetches the specified file from the plugin's data folder.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| `$filename`| string              | Yes      | The name of the file to retrieve the data from.                                            |

## Return 

* `mixed` Returns the data from the specified file. 

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
        $data = $this->getPluginData('example.txt');
    }
}
```

