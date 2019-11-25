# Plugin Method Overview

Each plugin extends TYPEMILL's base `Plugin` class. This base class provides some built-in methods that are useful for plugin developers. You can access all these helper methods with the keyword `$this`. The `$this` keyword simply references the object itself. For example:

````
public function myPluginMethod()
{
 	$path = $this->getPath();
 	
 	if($path == 'admin')
 	{
      	// do something 
 	}
}
````

Here is a list of all helper methods that the `Plugin`-class provides:

[TOC]

## getParams

Get the parameters of a request. DO NOT use it to get post data from a form. Only use it to get the query parameterss of a URL.

````
$this->getParams();
````
BE VERY CAREFUL with this, because you get the raw input or query data. You are responsible for checking and validating this data before you use it. If you create some kind of form in your plugin (e.g. a contact form), then Typemill handles the input validation, and you can get this validated input with the method `getFormData`.

## getFormData

Get the data of a post request, for example, when the user submits a form on the front end. Typemill will validate and check this data, and store it in a file temporarily. To get the correct data, you have to enter the name of your plugin as a parameter.

````
$inputData = $this->getFormData($pluginname);
````

If Typemill thinks that data has been submitted by a bot, then it simply returns the string "bot". So you should always check for this string first before you try to work with the data.

## generateForm

Generates an input form based on the YAML configuration file, and returns the form HTML. You have to pass the name of your plugin. 

````
$form = $this->generateForm($pluginname);
````

We will create a separate tutorial for this in future. 

## returnJson

Return data in the JSON format. This is useful if you want to create a plugin with API functionality.

````
$this->returnJson($arrayWithData);
````

## returnJsonError

Return a JSON response with a 400 error.

````
$this->returnJsonError($arrayWithData);
````

## getSettings

Get the settings of the whole application:

````
$settings = $this->getSettings();
````

## getPluginSettings

Get the settings of a specific plugin:

````
$pluginSettings = $this->getPluginSettings($pluginname);
````

## addJS

With the addJS funtion, you can add an external or internal JavaScript resources.

If you want to add an external JavaScript file, then simply add the full URL like this:

````
$this->addJS('https://some-url.com/with/path/to/javascript.js');
````

If you want to add a local JavaScript file that lives in your plugin folder, then simply add a relative URL like this:

````
$this->addJS('/yourpluginfolder/subfolder/javascript.js');
````

## addCSS

The addCSS function works exactly in the same way as the addJS method. You can add an external or internal resource:

````
$this->addCSS('https://some-url.com/with/path/to/style.css');
$this->addCSS('/yourpluginfolder/subfolder/style.css');
````

## addInlineJS

With this function, you can add any kind of inline JavaScript to all templates.

````
$this->addInlineJS('alert("hello");');
````

Add the plain JavaScript without the `<script>` tag, because TYPEMILL will add the tag for you.

## addInlineCSS

With this function, you can add any kind of inline CSS to all templates.

````
$this->addInlineCSS('body{ background: #000; }');
````

All these functions only work if the template has implemented the Twig functions needed to render the styles and scripts:

````
{{ renderCSS() }} // This tag should be placed in the HTML header.
{{ renderJS() }} // This tag should be placed before the closing body tag.
````

Check the chapter about [theme design](/for-theme-developers) for more informations.

## activateVue

This will add the Vue library to your front end assets. Vue is used by the Typemill author interface, so feel free to use it in front end, too.

````
$this->activateVue();
````

## activateAxios

Axios is our favourite library for AJAX calls, and it's used by Typemill, so you can easily use it in your plugin or frontend, too. 

````
$this->activateAxios();
````

In the front end, you can use the constant variable `myaxios` which is already initialized with the baseUrl of the application, so you don't have to worry about that. You can make your calls with relative URLs.


## activateTachyons

Tachyons is a very lightweight library for atomic or functional CSS, similar to the popular Tailwind library, but a bit older and much smaller. We plan to rewrite the pretty chaotic CSS of Typemill with tachyons in future. If you want to use it for your front end (themes or plugins), just activate it with:

````
$this->activateTachyons();
````

## getPath

With this function, you can get the actual path. It returns a simple string.

````
$this->getPath(); // returns something like 'imprint' or '/home/imprint'
````

This function can be handy if your plugin is designed to only work for a certain path on a website. 

````
if($this->getPath() == 'imprint')
{
  // do something in your plugin.
}
````

## getRoute

This function is a bit more flexible than the getPath() function, because it returns the Slim URI object:

````
$this->getRoute();
````

You have access to all Slim methods for this object, which include:

- `getScheme()`
- `getAuthority()`
- `getUserInfo()`
- `getHost()`
- `getPort()`
- `getPath()`
- `getBasePath()`
- `getQuery()` (returns the full query string, e.g. `a=1&b=2`)
- `getFragment()`
- `getBaseUrl()`

Please refer to the [Slim documentation](https://www.slimframework.com/docs/objects/request.html#the-request-method) for more information.

## getDispatcher

With this helper function, you can get the the Symfony event dispatcher. 

````
$dispatcher = $this->getDispatcher();
````

The dispatcher is also passed into your subscriber methods as a third argument by default. 

## getTwig

We already worked with this little helper. It returns the Twig template engine, and you can use it to render your own templates and add some variables. For example:

````
$twig   = $this->getTwig();  // get the twig-object
$loader = $twig->getLoader();  // get the twig-template-loader
$loader->addPath(__DIR__ . '/templates'); // add your template

// now render the template with some variables in it.
$twig->fetch('/yourTemplate.twig', array('mykey' => 'myvalue'));
````

Please check out the Twig documentation to learn more.

## addTwigGlobal

You can also add Twig Globals that you can use in the frontend.

````
$this->addTwigGlobal('text', new Text());
````

This will add the Twig variable 'text', which you can use in your templates like this:

````
{{ text }}
````

## addTwigFilter

You can also add Twig filters in your plugin like this:

````
$this->addTwigFilter('rot13', function($string){
  return str_rot13($string);
});
````

You can use this in your template like so:

````
{{ rot13('this is a text') }}
````

## addTwigFunction

You can add a Twig function in your plugin like this:

````
$this->addTwigFunction('myName', function(){
  return 'My name is ';
});
````

And again, you can use this function in your template like this:

````
{{ myName() }}
````

Please read the [Twig-Documentation](https://twig.symfony.com/doc/2.x/) to learn more about this. 

## markdownToHtml

If you want to render some Markdown as HTML in your plugin, then you can use the markdownToHtml function like this:

````
$html = $this->markdownToHtml('## My Markdown Headline');
````
