# Add New Middleware

If you are not familiar with the concept of middleware, please read the [Slim framework documentation](https://www.slimframework.com/docs/v3/concepts/middleware.html) first. With middleware, you can add some logic that is added to the live cycle of the application. Some examples of uses for middleware are:

* Authenticate a user.
* Validate user input.
* Add an error handler.

The concept of middleware is a bit harder to understand, but to add middleware with a plugin is pretty easy with the method `addNewMiddleware()`:

````
public static function addNewMiddleware()
{
    return array(
        'classname' => 'Plugins\MyPlugin\MyMiddleware', 
        'params' => false
    );
}

````

The method returns an array again and accepts two values:

* **classname**: The fully qualified name of the class, that should be called.
* **params**: False or an array.

You can create a new file called `MyMiddleware.php` in your plugin and add a middleware logic like this:

````
<?php

namespace Plugins\MyPlugin;

class MyMiddleware 
{   
    public function __invoke(\Slim\Http\Request $request, \Slim\Http\Response $response, $next)
    {
        // do something here 
        return $next($request, $response);      
    }
}
````

The support for middleware in Typemill is pretty basic right now, and it has some limitations. The most important limitation:

* Right now you can only add global middleware. You cannot add middleware only to specific routes.
* The order in which the middleware is executed is fixed, and you cannot manipulate it. This means that all the plugin middleware is executed after the TYPEMILL middleware. And the order depends on when your plugin gets loaded.

The middleware support in TYPEMILL will be improved in the future. For now, only use it if you really know what you wanna do. You can also add a new issue in GitHub, if you need a specific feature.

