#  storePluginYamlData

The method `$this->storePluginYamlData(string $filename, array $data)` is used to store YAML formatted data for the plugin. It validates the provided data against the plugin's field definitions and then updates the YAML file with the new data.

## Parameter

| Parameter | Type   | Required | Description                                      |
|-----------|--------|----------|--------------------------------------------------|
| $filename | string | Yes      | The name of the file to store the YAML data.    |
| $data     | array  | Yes      | The YAML data to be stored.                      |

## Return 

* `true` if the data is successfully stored. 
* `string` error message if an error occurs during the storage process. 
* `array` of validation errors if the provided data does not pass validation.

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
       $this->storePluginYamlData('config.yaml', ['key' => 'value']);
    }
}
```

