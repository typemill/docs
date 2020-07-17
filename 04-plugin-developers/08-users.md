# User Rights and Roles

Typemill uses an access control list (ACL) to manage user roles and access rights. You can use that system in many ways, for example:

* Change the permissions of an existing role.
* Add a new user role.
* Add a new field to the user profile page depending on the user role.
* Add a new route (url) and protect that route to user-roles with certain permissions.
* Add a new resource (e.g. a new navigation item to the admin settings) and only allow a new role to see it.

The ACL is implemented with the [laminas-permissions-acl-library](https://docs.laminas.dev/laminas-permissions-acl/) (former Zend Framework). If you want to work with user roles, then it is highly recommended to read the short documentation (5 pages).

## Standard User Roles and Privileges

Per default Typemill provides these user roles: 

* Member (can only edit his account data).
* Author (additionally can create and update his own content)
* Editor (additionally can publish, unpublish and delete all content).
* Administrator (can do everything). 

The roles are generated with a simple array and dispatched with the event `onRolesPermissionsLoaded`. You can get the user roles and user rights in your plugin like this:

````
<?php
namespace plugins\myplugin;
use \typemill\plugin;
class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
    	return array(
    		'onRolesPermissionsLoaded'		=> 'onRolesPermissionsLoaded'
    	);
    }
    public function onRolesPermissionsLoaded($rolesAndPermissions)
    {
        $rolesPermissions = $rolesAndPermissions->getData();
        print_r($rolesPermissions);
    }
}
````

This will output something like this:

````
Array
(
    [member] => Array
        (
            [name] => member
            [inherits] => 
            [permissions] => Array
                (
                    [user] => Array
                        (
                            [0] => view
                            [1] => update
                            [2] => delete
                        )
                )
        )
    [author] => Array
        (
            [name] => author
            [inherits] => member
            [permissions] => Array
                (
                    [mycontent] => Array
                        (
                            [0] => view
                            [1] => create
                            [2] => update
                        )
                    [content] => Array
                        (
                            [0] => view
                        )
                )
        )
    [editor] => Array
        (
            [name] => editor
            [inherits] => author
            [permissions] => Array
                (
                    [mycontent] => Array
                        (
                            [0] => delete
                            [1] => publish
                            [2] => unpublish
                        )
                    [content] => Array
                        (
                            [0] => create
                            [1] => update
                            [2] => delete
                            [3] => publish
                            [4] => unpublish
                        )
                )
        )
)
````

The logic is as simple as that: 

* user roles (e.g. "editor") 
* have privileges (e.g. "create") 
* to resources (e.g. "content"). 

User roles can also inherit the privileges of other user roles. For example, the "author" inherits the privileges of the "member", so he can also view, update and delete his own user-data.

## Change Permissions or Add User Roles

Yopu can change an existing user role or add a new user role. Let us add a new user role called "superauthor" who can not only view, create and update his own content, but also delete, publish and unpublish his own content:

````
public function onRolesPermissionsLoaded($rolesAndPermissions)
{
    $rolesPermissions = $rolesAndPermissions->getData();
    $superauthor = [
         'name' => 'superauthor',
         'inherits' => 'author',
         'permissions' => [
               'mycontent' => ['delete', 'publish', 'unpublish']
         ]
    ]
    $rolesPermissions['superauthor'] = $superauthor;
    $rolesAndPermissions->setData($rolesPermissions);
}
````

If you inherit the user role "author", then you don't have to repeat all the permissions. Simply add the new permissions and the ACL library will do all the rest.

## Resources

By default, Typemill uses four resources:

* `user` : my own user.
* `userlist`: all users.
* `mycontent`: my own content.
* `content`: all content.
* `system`: all kind of system settings and configurations, including plugins, themes and more.

In combination with the privileges (permissions), you can control the access to resources. In the example above the role `author` has the privilege to `view` the resource `content`, but he cannot do anything with the content like `update`, `delete`, `publish` or `unpublish`.

You have access to the resources in your plugin with the event `onResourcesLoaded`:

````
<?php
namespace plugins\myplugin;
use \typemill\plugin;
class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
    	return array(
    		'onResourcesLoaded'		=> 'onResourcesLoaded'
    	);
    }
    public function onResourcesLoaded($resources)
    {
        $allresources = $resources->getData();
        print_r($allresources);
    }
}
````

In most cases you don't want to add a new resource. But in some situations it is very useful, for example if you want to add a new item to the [system navigation](/plugin-developers/system-navigation).

