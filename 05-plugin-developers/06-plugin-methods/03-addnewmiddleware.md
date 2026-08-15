# addNewMiddleware

With the static method `addNewMiddleware()` you can add new middleware to the Typemill application. Read the [example to implement new middleware](/plugin-developers/examples/middlware) for details on how this works. 

## Example Usage

```php
public static function addNewMiddleware()
{
    return array(
        'classname' => 'Plugins\MyPlugin\MyMiddleware', 
        'params' => false
    );
}

```

