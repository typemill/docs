# Quick Start

Plugins are versatile tools that enhance Typemill by adding new features. You can create plugins using pure PHP or by leveraging a minimal PHP skeleton along with powerful Vue applications.

The core concept is straightforward: listen for one or more [events](/plugin-developers/plugin-events) triggered by Typemill, and then use specific [methods](/plugin-developers/plugin-methods) to introduce new functionality. This guide provides a quick overview of how to write a plugin using PHP. If you're interested in integrating a Vue application, please refer to the [quick start guide for Vue](/plugin-developers/quick-start-with-vue).

## Minimal Plugin

A basic plugin contains a php-file with the plugin class and a yaml-file with configurations. Both files must share its names with the plugin folder. For example:

````
/myplugin
  - myplugin.php
  - myplugin.yaml
````

A minimal yaml-file can just contain some basic information like this:

```yaml
name: My Plugin
version: 1.0.0
description: A short description.
author: Your Name Here
homepage: https://mycreatorhomepage.io
license: MIT
```

A minimal php file contains a plugin class that subscribes to at least one event and contains one subscriber method:

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

The basic tools and concepts that you need for a plugin are **namespaces**, **inheritances**, **events**, **methods**, **routes** and **middleware**.

## Namespace

All plugins use the namespace `plugins\`, followed by the name of your plugin.

## Extend the Typemill Plugin Class

Each plugin `extends` the base plugin class `\typemill\plugins`. 

```php
<?php

namespace plugins\myplugin;

use \typemill\plugin;

class myplugin extends plugin
{
}

```

When a class extends another class, it means that the new class automatically has access to the properties and methods defined in the parent class (inheritance). You can check all the properties and methods that are available for your plugin in the [method overview](/method-overview).

## The Typemill Event System

The Typemill plugin system adheres to the principles of event-driven development, utilizing the event dispatcher from Symfony. If you're familiar with Symfony's event dispatcher, you'll find the Typemill system intuitive.

If you are not familiar with events, then take a minute to grab the concept. When Typemill gets a request to render a page, it loads the settings, loads the plugins, loads the themes, creates the navigation, gets the page content, and finally renders the page. For each step, it fires an event and sends data with the events. Plugins can subscribe to these events, read, manipulate, and often return the manipulated data to Typemill.

For example, if Typemill has created the navigation, it fires the event "onNavigationLoaded" and sends the navigation data. If a plugin has subscribed to this event, it can read the navigation, change the navigation, and return it to Typemill.

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

## Methods

You can name the methods of your plugin class however you want. Many people give their methods the same name as the events. This way you can easily see which method is called by which event; but it's a matter of taste:

````
public static function getSubscribedEvents()
{
    return array(
        'onSettingsLoaded' => 'onSettingsLoaded'
    );
}

public function onSettingsLoaded($settings)
{
    // do something with the $settings
}

````

Within your methods, you can write your business logic.

## Event Arguments

All events pass some arguments into your method, and in many cases they also pass data that you can use or manipulate (like the $navigation in the example above). You can find all the details in the [event overview](/for-plugin-developers/documentation/event-overview).

## Add Routes

It is a common use case to add a new route, for example, to provide a new API endpoint, to receive and process data from a public form, or to add a new item to the system navigation and inject a new Vue application. It is not very hard and you can read more about it in the [example about routes](/plugin-developers/examples/routes).

## Add Middleware

In some cases, you may want to add middleware. The concept of middleware is widespread but a bit harder to grasp. Please refer to the [example about middleware](/plugin-developers/examples/middleware) if you want to know more details.

