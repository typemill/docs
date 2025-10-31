# Introduction to Events

The Typemill plugin system adheres to the principles of event-driven development, utilizing the event dispatcher from Symfony. During the application life cycle, Typemill consistently fires events and often sends data with these events. Developers can listen to these events, hook into the system, and add their own functionality.

## Subscribe to Events

Every plugin must have a public static method called `getSubscribedEvents()`. It returns an array with events as keys, and plugin methods as values.

```php
<?php

namespace plugins\myplugin;

use \typemill\plugin;

class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'onPagetreeLoaded' => 'onPagetreeLoaded'
        ];
    }

    public function onPagetreeLoaded($navigation)
    {
        // do something with the $navigation
    }
}

```

You can listen to several events in your plugin class:

````
public static function getSubscribedEvents()
{
  return [
        'firstEvent' => 'firstMethod',
        'secondEvent' => 'secondMethod'
    ]
}
````

You can also add several methods to a single event, and arrange these methods by priority:

````
public static function getSubscribedEvents()
{
    return array(
        'firstEvent' => array(
            array('firstMethod', 10),
            array('anotherMethod, 1)
        ),
        'secondEvent' => 'secondMethod'
    )
}
````

The rule for the order is pretty simple: The higher the order, the earlier the call. You can also use negative numbers like `-10` to give a method call a really low priority.

## Get and Return Event-Data

If Typemill passes data to your subscriber method, then you can get the data, use the data, manipulate the data, and return the data to the app. You can do this with two simple methods: `getData()` and `setData()`.

Let's take the HTML event as an example:

````
use Typemill\Plugin;

class MyPlugin extends Plugin
{
    public static function getSubscribedEvents()
    {
        return array(
            'onHtmlLoaded'      => 'onHtmlLoaded'
        );
    }

    public function onHtmlLoaded($html)
    {
        $data = $html->getData();
        $data .= '<p>This is a paragraph that I added at the end of the page content</p>';      
        $html->setData($data);
    }
}
````

## Other Event-Parameters

Typemill uses the Symfony event dispatcher for the event system. The event dispatcher adds two other variables to each event by default:

* The second parameter is the **name of the event**.
* The third parameter is the **event dispatcher** itself.

````
public function onHtmlParsed($html, $eventName, $dispatcher)
{
    // read the $eventName
    // use the $dispatcher
}
````

You can use the dispatcher variable to fire your own event. Another way to access the dispatcher is to use the method `$this-getDispatcher()` to access the dispatcher object from the dependency container.

## Application Lifecycle

The life cycle of Typemill can vary based on specific contexts. For example: The admin area does not fire events for content rendering since the content is not rendered at all in this context.

These events are fired in all cases: 

* `onSettingsLoaded` 
* `onPluginsLoaded`
* `onSessionSegmentsLoaded`
* `onRolesPermissionsLoaded`
* `onResourcesLoaded`
* `onTwigLoaded`

These events are only fired for system area:

* `onSystemnaviLoaded` 

These events are only fired for content pages:

* `onShortcodeFound`
* `onPagetreeLoaded`
* `onBreadcrumbLoaded`
* `onItemLoaded`
* `onMarkdownLoaded`
* `onMetaLoaded`
* `onRestrictionsLoaded`
* `onContentArrayLoaded`
* `onHtmlLoaded`
* `onPageReady`

