#  addNewRoutes

With the static method `addNewRoutes()` you can add new routes for frontend pages or API endpoints. Read the [example to implement new routes](/plugin-developers/examples/routes) for details on how this works. 

## Example Usage

```php
public static function addNewRoutes()
{
    return array(
        'httpMethod'    => 'get', 
        'route'         => '/myroute', 
        'class'         => 'Plugins\Myplugin\MypluginController:index'
    );
}
```

