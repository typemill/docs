# onRolesPermissionsLoaded

This event is triggered after the roles and permissions for ACL have been loaded. The event is useful if you want to add another user role.

## Availability

This event is everywhere available.

## Data

Plugins **must** return the array with roles and rights for this event; otherwise, the acl system will not work correctly.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Multi-dimensional array of roles and permissions. | 
| setData | array | required | Original or manipulated array with roles and rights. |

## Example Data

```
Array
(
    [guest] => Array
        (
            [name] => guest
            [inherits] => 
            [permissions] => Array
                (
                    [account] => Array
                        (
                            [0] => none
                        )

                )

        )

    [member] => Array
        (
            [name] => member
            [inherits] => 
            [permissions] => Array
                (
                    [account] => Array
                        (
                            [0] => read
                            [1] => update
                            [2] => delete
                        )

                )

        )
     ...
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
            'onRolesPermissionsLoaded' => 'onRolesPermissionsLoaded'
        ];
    }

    public function onRolesPermissionsLoaded($rolesAndPermissions)
    {
        $rolesPermissions = $rolesAndPermissions->getData();

        $provider = [
             'name' => 'provider',
             'inherits' => 'member',
             'permissions' => [
                'account' => [
                    'read','delete', 'update' 
                ]
             ]
        ];

        $rolesPermissions['provider'] = $provider;

        $rolesAndPermissions->setData($rolesPermissions);
    }
}
```

## See also

* [onResourcesLoaded](/plugin-developers/plugin-events/onresourcesloaded) event.
* [Example code to add a new user role](/plugin-developers/examples/add-new-userrole) for plugin developers.
* [User management](/admin-guide/users) for admins.

