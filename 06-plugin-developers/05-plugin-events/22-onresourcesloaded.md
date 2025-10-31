# onResourcesLoaded

This event is triggered after the resource-definitions have been loaded. The event is useful if you want to add another resource for user rights in acl.

## Availability

This event is everywhere available.

## Data

Plugins **must** return a resource-array for this event; otherwise, the system of rights and roles will not work correctly.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | One dimensional array of resource-names. | 
| setData | array | required | The original or manipulated resources. |

## Example Data

```
Array
(
    [0] => content
    [1] => mycontent
    [2] => account
    [3] => user
    [4] => system
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
            'onResourcesLoaded' => 'onResourcesLoaded'
        ];
    }

    public function onContentArrayLoaded($plugindata)
    {
        $resources = $plugindata->getData();

        $resources[] = 'myresource'

        $plugindata->setData($resources);
    }
}
```

## See also

* [onRolesPermissionsLoaded](/plugin-developers/plugin-events/onrolespermissionsloaded) event.
* [Example code to add a new user role](/plugin-developers/examples/add-new-userrole) for plugin developers.
* [User management](/admin-guide/users) for admins.

