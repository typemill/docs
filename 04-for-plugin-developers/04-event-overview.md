# The TYPEMILL Events

When a user visits an URL, there are a lot of steps taken to generate and finally return the website. Theses steps are called the life cycle. TYPEMILL has its own life cycle. When someone requests a page, then TYPEMILL initiates the application, loads all plugins, merges all settings, starts the template engine, scans the content folder, collects the content, and finally renders the page.

When TYPEMILL goes through this life cycle, it constantly fires events and often sends some data with these events. Developers can listen to these events, hook into the system, and add their own functionality. 

This is a list of all events that TYPEMILL fires during the life cycle. The order of the events follows the life cycle. In the last column, you can find the data that each event passes to your subscriber method.

| Event Name           | Description                              | Data                                 |
| -------------------- | ---------------------------------------- | ------------------------------------ |
| onPluginsLoaded      | TYPEMILL has loaded all plugins.         | No data                              |
| onSettingsLoaded     | TYPEMILL has loaded and merged all settings. This includes the basic app settings, all plugin settings, and the individual user settings. | Settings (a slim-object)             |
| onSessionSegmentsLoaded      | TYPEMILL has loaded all all session segments, which should start a session.         | Empty Array (array) |
| onTwigLoaded         | TYPEMILL has loaded the template engine Twig. | No data                              |
| onPagetreeLoaded     | TYPEMILL has scanned the content folder, and has generated the content structure. | Content structure (array of objects) |
| onBreadcrumbLoaded   | TYPEMILL has created a breadcrumb for the page. | Breadcrumb (array)                   |
| onItemLoaded         | TYPEMILL has created the page item.      | Item (object)                        |
| onMarkdownLoaded     | TYPEMILL has loaded the page content.    | Page content (markdown syntax)       |
| onContentArrayLoaded | TYPEMILL has transformed the page content into an array. | Page content (array)                 |
| onHtmlLoaded         | TYPEMILL has transformed the Markdown content to HTML (with the Parsedown library). | Page content (html syntax)           |
| onPageReady          | TYPEMILL has collected all data for the page, and will send it to the template in the next step. | All page data (array)                |
| onPagePublished     | TYPEMILL has published a content page.    | Page item (obejct)       |
| onPageUnpublished     | TYPEMILL has unpublished a content page.    | Page item (object)       |
| onPageDeleted     | TYPEMILL has deleted a page.    | Page item (object)       |
| onPageSorted     | TYPEMILL has sorted/moved a page.    | Input Params with old and new position (array)       |


If TYPEMILL passes data to your subscriber method, then you can get the data, use the data, manipulate the data, and return the data to the app. You can do this with two simple methods: `getData()` and `setData()`.

Let's take the HTML event as an example:

````

class MyPlugin extends \Typemill\Plugin
{
    public static function getSubscribedEvents()
    {
		return array(
			'onHtmlLoaded' 		=> 'onHtmlLoaded'
		);
    }

	public function onHtmlParsed($html)
	{
		$data = $html->getData();

		$data .= '<p>This is a paragraph that I added at the end of the page content</p>';		

		$html->setData($data);
	}
}
````

TYPEMILL uses the Symfony event dispatcher for the event system. The event dispatcher adds two other variables to each event by default:

* The second parameter is the **name of the event**.
* The third parameter is the **event dispatcher** itself.

So in each of your event methods in your plugin, you can also read the event name, and you have access to the dispatcher object itself:

````
public function onHtmlParsed($html, $eventName, $dispatcher)
{
	// read the $eventName
	// use the $dispatcher
}
````

There are not many use cases for accessing the event name or the dispatcher in this way. Theoretically, you could fire your own events with the dispatcher object, but it'ss much better to access the dispatcher object within Slim's dependency container.

The dependency container is one of the properties and methods provided by TYPEMILL's basic plugin class. Because all plugins extend this basic plugin class, all plugins have access to these useful properties and methods.

We'll learn  more about that in the next chapter.