# Configuration Forms for Themes and Plugins

With configurations, you can add a lot of flexibility to your plugins and themes. And with forms, you can enable the authors to edit those configurations and to adapt the plugins and themes to his needs.

[TOC]

## Example for a Form Definition

To add a configuration form to your plugin or theme, just add some form-definitions to the YAML-configuration file of your theme or plugins. All form definitions start with the keywords `forms` and `fields` followed by the definition of the fields: 

````
name: Example Plugin
version: 1.0.0
description: Add a short description
author: Firstname Lastname
homepage: http://your-website.net
licence: MIT

forms:
  fields:
    layout:
      type: select
      label: Select a Layout
      options:
        edgeless: Edgeless
        block: Block
        classic: Classic
````

This will add a selectbox with the three options and the name "layout". You can use all fields from the field overview.

## Storage Configurations

If the user fills out the configuration form of your plugin or theme and clicks on save, then the configuration will be stored in the `settings.yaml`-file that you can find in the folder `settings`. 

If you use the example above for a theme called "exampletheme", then Typemill will store something like this in the settings.yaml-file:

```
...
  themes:
    exampletheme:
      layout: Edgeless
...
```

If you use a plugin called "exampleplugin", then it looks like this: 

```
...
  plugins:
    exampleplugin:
      layout: Edgeless
...
```


## Use Configurations

You can use all stored configuration for a theme in your twig-templates with the `settings`-variable:

```
{{ settings.themes.exampletheme.layout }} 
```

This will print out "Edgeless".

In your plugins you can use the stored values with the event `onSettingsLoaded`:

```
<?php

namespace Plugins\Example;

use \Typemill\Plugin;

class Example extends Plugin
{	
    private $settings; 

    public static function getSubscribedEvents()
    {
		return array(
			'onSettingsLoaded'		=> 'onSettingsLoaded'
		);
    }
	
    # when settings are loaded
	public function onSettingsLoaded($settings)
	{
        # get settings, so you can use them in all methods
		$this->settings = $settings->getData();

        # or use them directly in this method
        $layout = $this->settings['settings']['plugins']['exampleplugin']['layout'];
	}

```

## Play Around

As you can see, everything follows a simple logic and is easy to use. If you did not get it yet, no problem! Simply download a plugin or a theme, check the form configurations in the yaml-files and play around with it.
