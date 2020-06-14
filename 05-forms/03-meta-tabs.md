# Forms for Page Meta-Tabs

Do you need a special meta-information for pages in your theme? Or do you want to create a SEO-tab for each page? No problem, because you can add your own meta-fields or even your own meta-tabs with a plugin and with the help of vue.js.

[TOC]

## Add Fields to a Tab

By default, each page has a meta-tab called "meta". To add a new field to that tab, simply create a plugin or a theme with a new YAML definition, that starts with the keyword `metatabs` followed by the name of the tab `meta` like this:

```
name: Example Plugin
version: 1.0.0
description: Add a short description
author: Firstname Lastname
homepage: http://your-website.net
licence: MIT

metatabs:
  meta:
    fields:
      myfield:
        type: select
        label: Select an Option
        options:
          green: Green
          blue: Blue

```

This will add a new select-field to the existing tab "meta".

## Store and Use Meta-Information

Similar to the famous "frontmatter", Typemill can store any kind of meta-information for a each content page. Different to frontmatter, Typemill does not store the meta-information in the markdown-file, but it creates a separate yaml-file to keep the markdown-file clean and readable.

Typemill uses some standard-meta-information like the title, the description and the author. But you can also add your own fields to the meta-tab and use all meta-information in your twig-templates with the variable `metatabs` like this:

```

{{ metatabs.meta.myfield }}

```

You can also access the information in your plugin with the event `onMetaLoaded` like this:

```
<?php

namespace Plugins\Example;

use \Typemill\Plugin;

class Example extends Plugin
{	
	public static function getSubscribedEvents()
	{
		return array(
			'onMetaLoaded'		=> 'onMetaLoaded'
		);
	}
	
	public function onMetaLoaded($meta)
	{
		$meta = $meta->getData();
       
        if($meta['meta']['myfield'] == "green")
        {
            # make something green here
        }
}

```

## Create a New Meta-Tab

With a plugin, you can also create a new meta-tab for pages. Just add a new yaml-configuration to your plugin and give your meta-tab a name like this:

```
name: Example Plugin
version: 1.0.0
description: Add a short description
author: Firstname Lastname
homepage: http://your-website.net
licence: MIT

metatabs:
  mytab:
    fields:
      myfield:
        type: text
        label: Add some text

```

If you activate the plugin, then you will already see a new tab called "mytab" for each page. 

However, if you click on the tab, you will not see any forms yet. The reason is quite simple: Different to forms for plugins, themes or public pages, the forms for meta-tabs are rendered with Vue.js and not with PHP. In order to render the forms, you have to activate the vue form generator. This is done with a simple copy & paste.

## Add the Vue Component

You can activate the vue form-generator in your plugin with a simple script (a vue-component). In the first step we add that script to the editor like this:

```
<?php

namespace Plugins\Example;

use \Typemill\Plugin;

class Example extends Plugin
{	
    public static function getSubscribedEvents()
    {
		return array(
			'onTwigLoaded'		=> 'onTwigLoaded'
			'onMetaLoaded'		=> 'onMetaLoaded'
		);
    }
	
	public function onTwigLoaded()
	{
		$this->addEditorJS('/example/js/editor.js');
	}

	public function onMetaLoaded($meta)
	{
		$meta = $meta->getData();
        
        # do something with the fields:
        $myTabInformation = $meta['mytab'];
	}

}

```

We used the event `onTwigLoaded` to add the script, but you can also use other events if you want. 

In the next step we will create the script. The script is a simple vue-component that looks always like this:

```
Vue.component('mytab', {
	props: ['saved', 'errors', 'formdata', 'schema'],
	template: '<section><form>' +
				'<component v-for="(field, index) in schema.fields"' +
            	    ':key="index"' +
                	':is="selectComponent(field)"' +
                	':errors="errors"' +
                	':name="index"' +
                	'v-model="formdata[index]"' +
                	'v-bind="field">' +
				'</component>' + 
				'<div v-if="saved" class="metaLarge"><div class="metaSuccess">Saved successfully</div></div>' +
				'<div v-if="errors" class="metaLarge"><div class="metaErrors">Please correct the errors above</div></div>' +
				'<div class="large"><input type="submit" @click.prevent="saveInput" value="save"></input></div>' +
			  '</form></section>',
	methods: {
		selectComponent: function(field)
		{
			return 'component-'+field.type;
		},
		saveInput: function()
		{
  			this.$emit('saveform');
		},
	}
})
```
You can simply **copy and paste** this component, because the vue-component will always look the same. Just change the name of the component `Vue.component('mytab', {})` to the name of your tab. Please use an **unique name** for your tab, because there might be conflicts if another plugin uses the same name for a tab.

This is what the vue component does:

* It loops over the fields that you defined in your YAML file.
* It dynamically adds the field component according to the type of each field.
* It binds all neccessary data like the field-definitions, the errors, the name of the field and so on.
* It adds a save button and messages for success and error.

If you want to know how it works in detail, then check the file `vue-meta.js` in the folder `system/author/js`.

## Extend the Vue Component

The vue-component above just renders the forms and adds a store-button. If you are a vue-developer, then you can do much more and extend the vue-component with all kind of functionality that you want. Imagine a seo-tab that fetches some statistics for the page from the API of the Google Search Console and displays some charts. I did not try this yet, but it is no problem for you, I guess.

## Limitations

The meta-tabs are flexible as hell, but the actual vue-form-builder has some limitations compared to public forms or to configuration-forms for plugins and themes:

* You cannot group the fields with **fieldsets**.
* You cannot use the fieldtype **hidden**.
* You cannot add text with the field-type **paragraph**.
* You cannot use the **help** text.
* You can use attributes like **required** or **pattern** in your yaml-definition. They are used for the backend validation, but they have no effect in the frontend.

