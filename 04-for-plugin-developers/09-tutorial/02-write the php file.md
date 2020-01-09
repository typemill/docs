# Write the Basic PHP-File

We have our basic configuration file, so let's start with our basic PHP file now. As mentioned before, the basic PHP file must have the same name as the plugin folder:

```
/cookieconsent
  - cookieconsent.yaml
  - cookieconsent.php
```

TYPEMILL uses object oriented PHP, so let us create a class for our plugin.

## Write The Plugin Class

Open the `cookieconsent.php` file, and add the following code:

```
<?php

namespace plugins\cookieconsent;

use \typemill\plugin;

class cookieconsent extends plugin
{
   ...
}
```

Instead of `use \typemill\plugin;`, you can also write:

```
class cookieconsent extends \typemill\plugin
{
  ...
}
```

 Let's go over what is happening here:

- **namespace**: In the first line, we define the namespace for our plugin. The namespace always starts with `plugin\`, followed by the plugin name, so it is `plugin\cookieconsent`. Again: the namespace must have the same name as the folder and the file of your plugin.
- **use namespace**: In the next line, we import the namespace for the plugin class of TYPEMILL with `Use \typemill\plugins`.
- **class**: In the next line, we create the plugin class. Again: the class must have the same name as the plugin, so it is `class cookieconsent`.
- **extends**: The class always extends the plugin base class of TYPEMILL, so it is `extends plugin`  or, if you did not import the namespace with use before, then `extends \typemill\plugin`.

In case you are not familiar with the keyword `extend`, this means, that your plugin class extends the plugin class of TYPEMILL, and this means, that your plugin class will incorporate a lot of useful methods from that TYPEMILL plugin class. We will work with these useful helper methods later on in this tutorial.

## Add The First Method

Now we will add the first method. **Every plugin** must provide a static method called `getSubscribedEvents`. The method looks like this:

````
<?php

namespace plugins\cookieconsent;

use \typemill\plugin;

class cookieconsent extends plugin
{
    public static function getSubscribedEvents()
    {
    	return array(
    		'onSettingsLoaded'		=> 'onSettingsLoaded',
    		'onTwigLoaded' 			=> 'onTwigLoaded'
    	);
    }
}
````

As you can see, the method returns a simple array. And the name `getSubscribedEvents` indicates, that this has something to do with events. So it's time to talk about events now.

## Introducing Events

Nearly all content management systems use events to create a plugin system. So if you know what events are, then you can create plugins for all these systems much more easily. But what are events in PHP?

If you have ever written some code in JavaScript, then you know, what events are. If not: An event system follows a pretty simple IFTTT-logic like: "If This Than That". If this happens, than do that. If that happens, then do this. 

As a plugin developer, you do exactly the same: When someone visits the URL of a TYPEMILL website, then TYPEMILL creates the page in several steps. For each step, TYPEMILL fires a special event. As a developer, you 'listen' or 'subscribe' to some of these events, and if the event happens, then you tell the system to perform an action, or more specifically, to call a method within your plugin code. That is basically the whole magic trick.

## Cookie Consent Event Logic

Let's describe some IFTTT logic for the Cookie Consent plugin. It is as simple as this:

* If the system has loaded all settings, then give me the settings.
* If the system has loaded the template engine, then 
  * add a CSS-file,
  * add a JavaScript-file and
  * inject a little sub-template with the initial JavaScript into the theme and 
  * read the user settings and paste the values for colors and content into the script.

## Event Logic in an Array

Event systems work with arrays, and it is pretty easy to describe IFTTT logic with an array like this:

* "If this happens": That is the event. We use the event name as the key in our array.
* "Then do that": That is the plugin method. We use the name of the plugin method as the value in our array.


So when the event `onSettingsLoaded` is fired, then call the method `onSettingsLoaded` in our plugin code:

````
array('onSettingsLoaded' => 'onSettingsLoaded');
````

You can name your method in your plugin however you want, but it is good practice to give the method the same name as the event. As you can see, working with events is pretty simple.

## Add Another Method

To test this out, simply add a method called `onSettingsLoaded`, and add some logic to the method:

````
class cookieconsent extends plugin
{
	public static function getSubscribedEvents()    
	{        
		return array('onSettingsLoaded' => 'onSettingsLoaded');    
	}
	
	public function onSettingsLoaded()
	{
		die('sorry to interrupt you!');
	}
}
````

You can also add several methods to one event, and you can even set the order in which the methods should be called. This is done by a multi-dimensional array like this:

````
public static function getSubscribedEvents()
{
  	return array(
  		'onSettingsLoaded' => array(
  			array('myFirstMethod', 10),
  			array('mySecondMethod', -10)
  		)
  	);
}
````

The method with the priority of  `10`  gets called first, the method with the priority of `-10` gets called last.

Just to complete your vocabulary: In event-driven programming people talk about `listening` to events, or `subscribing` to events. That is why the static method has the name `getSubscribedEvents`.

And that is the magic of events!

## Event Overview

We will only use two events in our cookie consent plugin, but TYPEMILL fires (or "dispatches") many more events, which can be very useful if you want to create more complex plugins. You can find a list of all events in the [Event Overview in the documentation](/for-plugin-developers/documentation/event-overview).

## Next: Add Methods to your Plugin

We have subscribed to some events. Now it's time to add some logic to our plugin. Let's do this in the next chapter.

