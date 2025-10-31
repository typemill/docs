# System Navigation

Typemill provides five navigation items in the system navigation by default: 

* system
* themes
* plugins
* account
* users

A standard user will only see the "account" item, all other items require access rights for the resource "system" which is usually restricted to administrators.

You have access to the navigation items with the event `onSystemnaviLoaded`:

````
<?php
namespace plugins\myplugin;
use \typemill\plugin;
class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
        return array(
            'onSystemnaviLoaded'        => 'onSystemnaviLoaded'
        );
    }

    public function onSystemnaviLoaded($navidata)
    {
        $systemnavi = $navidata->getData();

        # print out the navigation-data to inspect the structure
        print_r($systemnavi);

        # send back to application
        $navidata->setData($systemnavi);
    }
}
````

With this event you can enhance the system navigation. You can add your own navigation items, restrict the item to a certain user right and add content/functionality to the opening page.

## New System Navigation for a User Role

Let's say that you want to create a new item in the system navigation called "My articles" and this navigation item should only be visible to users with the roles "author" and "editor". If you are not familiar with user rights yet, then read the chapter about [users](/plugin-developers/users) first.

At first we add a new resource called `myarticles`:

````
public function onResourcesLoaded($resources)
{
    $allresources = $resources->getData();
    $allresources[] = 'myarticles';
    $resources->setData($allresources);
}
````

Next we add this resource to the users "author" with the privilege to view that resource. The "editor" automatically has the same rights, because he inherits the privileges from the "author":

````
public function onRolesPermissionsLoaded($rolesAndPermissions)
{
    $rolesPermissions = $rolesAndPermissions->getData();
    $rolesPermissions['author']['permissions']['myarticles'] = ['view'];
    $rolesAndPermissions->setData($rolesPermissions);
}
````

Before we add a new clickable item to the system navigation, we want to define a new route (url), so that a new page opens if the user clicks on the navigation item. We do this with the static function `addNewRoutes`: 

````
    public static function addNewRoutes()
    {
        return [
           'httpMethod' => 'get', 
           'route' => '/tm/myarticles', 
           'name' => 'myarticles.show', 
           'class' => 'Typemill\Controllers\SettingsController:showBlank', 
           'resource' => 'myarticles', 
           'privilege' => 'view'];
    }
````

The method of the route is `get` and the route points to the url `/tm/myarticles`. We can use the name for the name `myarticles.show` to reference that route later. The route opens a standard class `Typemill\Controllers\SettingsController:showBlank`. This class should be used for all new navigation items. It simply handles the data for a new blank page in the system. Finally we restrict the access to the route to the resource `myarticles` with the privilege `view`. All users who have that privilege can see and click the new navi-item now. 

But wait: We don't see the item in the system navigation yet. So let us add the item quickly with the event `onSystemnaviLoaded`:

```php
    public function onSystemnaviLoaded($navidata)
    {
        $systemnavi = $navidata->getData();
        $systemnavi['My Articles'] = ['routename' => 'myarticles.show', 'icon' => 'icon-xyz', 'aclresource' => 'myarticles', 'aclprivilege' => 'view'];

        # if user visits the url, then set the navigation item active:
         if($this->getPath() == 'tm/myarticles')
         {
               $systemnavi['My Articles']['active'] = true;
        }
        $navidata->setData($systemnavi);
    }
```

We again add the resource and privilege here with `aclresource` and `aclprivilege`. This way we make sure that only the users with that permissions ("authors" and "editors") can see the navigation item. If you skip that, then the route is restricted to "authors" and "editors", but not the navigation item. This means that a "member" can see the navigation item, but if he clicks on the item, he will get an error or will be redirected to another page. So if you restrict a route, then you should also restrict the visibility of the navigation item.

You can test it easily: Login with the user role "member" first and you won't see the new navigation item. Then login as "author" or "editor" and you will see the navigation item.

 If you click the new item, then a blank page will open. The description how to add new functionality to the page will be added in future.

