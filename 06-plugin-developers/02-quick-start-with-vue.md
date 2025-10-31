#  Quick Start with Vue

Typemill is primarily built on PHP, but it operates mainly through an internal API and a Vue application for the user interface. While you will need some basic PHP code for a plugin, you can quickly switch to Vue and integrate applications in the author interface, in the admin interface, and in the frontend. With Vue, you can utilize all existing API methods or easily add your own API endpoints using PHP.

## Use Cases

There are three primary use cases for Vue applications:

1. **Editor Tabs**: Add a new tab within the editor to run the Vue application.
2. **System Menu**: Introduce a new navigation item in the system menu to run the Vue application.
3. **Frontend Page**: Embed the Vue application into a frontend page.

We will explore all these use cases in detail.

## Basic Plugin with PHP and YAML

Before diving into Vue, we need to establish a plugin structure using PHP and YAML as follows:

```
/myplugin
  - myplugin.php
  - myplugin.yaml
  /js
    - myapp.js
  /css
    - myapp.css
```

Next, populate the YAML file with some basic information:

```yaml
name: My Plugin
version: 1.0.0
description: A brief description.
author: Your Name Here
homepage: https://mycreatorhomepage.io
license: MIT
```

Now we can create a basic PHP structure and listen for specific events where we want to inject the Vue application:

```php
<?php

namespace plugins\myplugin;

use \typemill\plugin;

class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
        return array(
            'onTwigLoaded' => 'onTwigLoaded'
        );
    }

    public function onTwigLoaded()
    {
        if ($this->editorroute) {
            $this->addJS('/myplugin/js/myapp.js');
            $this->addCSS('/myplugin/css/myapp.css');
        }
    }
}
```

## Vue Apps in a Tab

Let us start with a Vue application that runs in a new tab that is visible at the top of each page. First, use PHP to inject a new JavaScript file in the editor interfaces as described above:

```php
    public function onTwigLoaded()
    {
        if ($this->editorroute) {
            $this->addJS('/myplugin/js/myapp.js');
            $this->addCSS('/myplugin/css/myapp.css');
        }
    }
```

The basic Vue app in the JavaScript should look like this:

```javascript
app.component('tab-mytab', {
    props: ['item', 'formData', 'formDefinitions', 'saved', 'errors', 'message', 'messageClass'],
    template: `<section class="dark:bg-stone-700 dark:text-stone-200">
                    <form>
                        <div v-for="(fieldDefinition, fieldname) in formDefinitions.fields">
                            <fieldset class="flex flex-wrap justify-between border-2 border-stone-200 p-4 my-8" v-if="fieldDefinition.type == 'fieldset'">
                                <legend class="text-lg font-medium">{{ fieldDefinition.legend }}</legend>
                                <component v-for="(subfieldDefinition, subfieldname) in fieldDefinition.fields"
                                    :key="subfieldname"
                                    :is="selectComponent(subfieldDefinition.type)"
                                    :errors="errors"
                                    :name="subfieldname"
                                    :userroles="userroles"
                                    :value="formData[subfieldname]" 
                                    v-bind="subfieldDefinition">
                                </component>
                            </fieldset>
                            <component v-else
                                :key="fieldname"
                                :is="selectComponent(fieldDefinition.type)"
                                :errors="errors"
                                :name="fieldname"
                                :userroles="userroles"
                                :value="formData[fieldname]" 
                                v-bind="fieldDefinition">
                            </component>
                        </div>
                        <div class="my-5">
                            <div class="block w-full h-8 my-1">
                                <transition name="fade">
                                    <div v-if="message" :class="messageClass" class="text-white px-3 py-1 transition duration-100">{{ $filters.translate(message) }}</div>
                                </transition>
                            </div>
                            <input type="submit" @click.prevent="saveInput()" :value="$filters.translate('save')" class="w-full p-3 my-1 bg-stone-700 dark:bg-stone-600 hover:bg-stone-900 hover:dark:bg-stone-900 text-white cursor-pointer transition duration-100">
                        </div>                      
                    </form>
                </section>`,
    methods: {
        selectComponent: function(type)
        { 
            return 'component-' + type;
        },
        saveInput: function()
        {
            this.$emit('saveform');
        },
    }
})
```

With this component, you can render forms that you define in the YAML file of your plugin like this:

```yaml
name: My Plugin
version: 1.0.0
description: A brief description.
author: Your Name Here
homepage: https://mycreatorhomepage.io
license: MIT
metatabs:
  mytab:
    fields:
      myfield:
        type: select
        label: Select an Option
        options:
          green: Green
          blue: Blue
```

Make sure that you use the name "tab-mytab" for your Vue component so that it corresponds with the name "mytab" that you defined in the YAML definition. Now Typemill will do everything automatically: It will render the form and store the form data in the YAML file of the page if the admin clicks on the save button. You can then use the form data in the frontend with Twig like this:

```twig
{{ metatabs.mytab.myfield }}
```

Read all about this kind of form-generation in the chapter about [forms and meta-tabs](/forms/meta-tabs).

If you do not need the form builder for your tab, then simply delete the whole for-loop and the methods and build something on your own. Check the ebook plugin or the SEO plugin to see what is possible.

## Vue Apps in the System Menu

You are not limited to meta-tabs, you can also create a new item in the system navigation and run your vue app there. It is a bit more complicated, but let's do it quickly. You can also check the example on how to [change the system navigation](/plugin-developers/examples/system-navigation) for more details. 

First we have to add a new navigation item with php like this: 

```php

    public function onSystemnaviLoaded($navidata)
    {
        $systemnavi = $navidata->getData();
        $systemnavi['myvueapp'] = ['routename' => 'myvueapp.show', 'icon' => 'icon-xyz', 'aclresource' => 'mycontent', 'aclprivilege' => 'view'];

        # if user visits the url, then set the navigation item active:
         if($this->getPath() == 'tm/myvueapp')
         {
               $systemnavi['myvueapp']['active'] = true;
        }
        $navidata->setData($systemnavi);
    }
```

This will add a new navigation item into the system navigation. Note that you can control the visibility of the navigation item with acl by defining the required resources and privileges. In this case, everybody who can edit content (starting from the author role) will see the navigation item. 

To make the navi item clickable, we need to [create a new route](/plugin-developers/examples/routes) like this: 

```php
    public static function addNewRoutes()
    {
        return [
          'httpMethod' => 'get', 
          'route' => '/tm/myvueapp', 
          'name' => 'myvueapp.show', 
          'class' => 'Typemill\Controllers\SettingsController:showBlank', 
          'resource' => 'mycontent', 
          'privilege' => 'view'
        ];
    }
```

Now we have an empty page in the system area. Now simply include the vue application if the admin clicks on the new navi-item, so we just add a single line to the event `onSystemnaviLoaded` like this: 

```php

    public function onSystemnaviLoaded($navidata)
    {
        $systemnavi = $navidata->getData();
        $systemnavi['myvueapp'] = ['routename' => 'myvueapp.show', 'icon' => 'icon-xyz', 'aclresource' => 'mycontent', 'aclprivilege' => 'view'];

        # if user visits the url, then set the navigation item active:
        if($this->getPath() == 'tm/myvueapp')
        {
               $systemnavi['myvueapp']['active'] = true;
               $this->addJS('/myplugin/js/myvueapp.js');
        }
        $navidata->setData($systemnavi);
    }
```

That's it. 

You can create any type of vue application, and you can also create any number of additional api endpoints that your vue application wants to use. Store data, create books, send data, fetch data from other apis, there are not many limits.

## Vue App in the Frontend

Documentation of this feature follows soon.

