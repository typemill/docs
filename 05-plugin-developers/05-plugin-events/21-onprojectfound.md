# onProjectFound

This event is triggered after a project has been set. This happens, if a project is detected in the url of the current page. The event is useful if your plugin should work with the projects feature.

## Availability

This event is only available if the project feature is activated and if the current url contains a project segment.

## Data

The method `setData` is not available for this event.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Array with the id of the project as value. | 

## Example Data

```
Array
(
   'project' => 'myproject'
)
```

## Example Usage

```php
<?php

namespace plugins\myplugin;

use \typemill\plugin;

class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'onProjectFound' => 'onProjectFound'
        ];
    }

    public function onProjectFound($projectData)
    {
        $this->project = $projectData->getData()['project'] ?? null;
    }

    private function deleteSearchIndex()
    {
        $index = 'index';

        if($this->project)
        {
            $index .= '_' . $this->project;
        }

        $storage = new StorageWrapper($this->settings['storage']);

        # delete the index file here
        $storage->deleteFile('dataFolder', 'bettersearch', $index . '.json');       
    }
}
```

## See also

* [Projects tab](/admin-guide/project-tab)
* [Projects for authors](/author-guide/multi-project)

