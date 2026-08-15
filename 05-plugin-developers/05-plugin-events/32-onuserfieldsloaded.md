# onUserfieldsLoaded

This event is triggered after the userfields have been loaded. The event is useful if you want to add another field for all users or a certain user role.

## Availability

This event is available where

## Data

Plugins **must** return a content-array for this event; otherwise, the content will not be available in the frontend.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | The `userrole`, the `inspectorrole`, and the `userfields`. | 
| setData | array | required | Must return all data. |

## Example Data

```
Array
(
    [userrole] => administrator
    [inspectorrole] => administrator
    [userfields] => Array
    (
        [username] => Array
            (
                [name] => username
                [label] => Username (read only)
                [type] => text
                [readonly] => 1
             )

        [image] => Array
             (
                [name] => profileimage
                [label] => Profile Image
                [type] => image
             )

        [description] => Array
             (
                [name] => description
                [label] => Author-Description (Markdown)
                [type] => textarea
             )
         ...
    )
)

```

## Extra Note

If you add a field, then the key of the field and the name of the field must be the same. Otherwise the validation will remove that field.

```php

# The KEY and the NAME must be the same "providerlogo"
$userfields['providerlogo'] = [
    'name'    => 'providerlogo',
    'label'   => 'Logo', 
    'type'    => 'image'
];

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
            'onUserfieldsLoaded' => 'onUserfieldsLoaded'
        ];
    }

    public function onUserfieldsLoaded($data)
    {
        $userforms = $data->getData();

        $userfields = $userforms['userfields'];

        if($userforms['userrole'] == 'provider')
        {
            # add validation rules
            $userfields['providerlogo'] = [
                  'name'    => 'providerlogo',
                  'label'   => 'Logo', 
                  'type'    => 'image'
            ];
            $userfields['providertext'] = [
                  'name'    => 'providertext',
                  'label'   => 'Small text for infobox', 
                  'type'    => 'textarea',
                  'regex'   => '/^[a-zA-Z0-9 .,?!;:_\-\*]*$/'
            ];
        }

        $userforms['userfields'] = $userfields;

        $data->setData($userforms);
    }
}
```

## See also

* [onRolesPermissionsLoaded](/plugin-developers/plugin-events/onrolespermissionsloaded) event.
* [onResourcesLoaded](/plugin-developers/plugin-events/onresourcesloaded) event.
* [Example code to add a new user role](/plugin-developers/examples/add-new-userrole) for plugin developers.
* [User management](/admin-guide/users) for admins.

