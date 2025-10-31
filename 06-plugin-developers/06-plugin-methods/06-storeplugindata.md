#  storePluginData

The method `$this->storePluginData($filename, $data, $method = NULL)` is used to store data for the plugin. It creates a folder with the name of the plugin inside the `data` folder and stores the provided data into the specified file.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| `$filename`| string              | Yes      | The full filename with filetype (e.g., "data.txt").                                      |
| `$data`    | string/array       | Yes      | The data to be stored.                                                                     |
| `$method`  | string              | No       | A method to transform the data into a writable format (e.g., `serialize` or `json_encode`). Defaults to null if not provided. |

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
        // Storing string data
        $this->storePluginData('data.txt', 'Some text data');

        // Storing array data with serialization
        $this->storePluginData('data.json', ['key' => 'value'], 'serialize');
    }
}
```

