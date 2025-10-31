# Plugin Configurations with Yaml

Every plugin for Typemill must have a plugin configuration file. This is quickly done with a small YAML-file, which must have the same name as your plugin folder. The YAML-file contains some basic informations and some optional configurations:

* **Basic information**: With a readable plugin name, a description, a version number, and more.
* **Settings**: Some default variables, that you want to use for your plugin.
* **Forms**: The definition of forms. Users can customize the plugin with these forms in the author panel.

## Add Basic Informations

The basic information in the YAML-file look like this:

```yaml
name: My Plugin
version: 1.0.0
description: A short description.
author: Your Name Here
homepage: https://mycreatorhomepage.io
license: MIT
```


The version number is critical for update checks and notifications, following a recommended schema like 1.2.3 (major.minor.bugfix).

## Add a Donation Button

You can also add a donation button for paypal like this:

```yaml
name: My Plugin
version: 1.0.0
description: A short description.
author: Your Name Here
homepage: https://mycreatorhomepage.io
license: MIT
paypal: https://paypal.me/yourname
amount: 10
```


The button will appear in the plugin configuration of the author panel. If your plugin is uploaded to the official [plugin-website](https://plugins.typemill.net) of Typemill, then a donation button will appear on the overview page and on the detail page. If you want to add your plugin to the plugin collection, please open a new issue [on GitHub](https://github.com/typemill/typemill).

## Default Settings

Sometimes you want to use default variables in your plugin, for example to add a default value for a button-label. With YAML you can easily do this: Just create a new block that starts with `settings`, and write all your settings as simple key-value-pairs. Indent them with two spaces like this:

```yaml
settings:
  key1: value1
  Key2: value2
  key3: value3
```


The settings are automatically merged with all other settings of Typemill.

## Forms

You can make your plugin variables editable for the user in the author panel. To do this, just add another block that starts with `forms` and `fields`. After that, you can define a wide range of input fields with YAML. It starts with the name of the field, followed by the field definition.

```
forms:
  fields:

    chapter:
      type: text
      label: chapter
      placeholder: Add Name for Chapter
      required: true

    start:
      type: text
      label: Start-Button
      placeholder: Add Label for Start-Button
      required: true
```


Typemill will use these definitions and generate input fields for the author panel on the fly, so that the user can edit the values and customize the plugin. If you have defined settings with the same name as the field name (e.g. `chapter`), then the input field in the author panel will automatically be prefilled with your settings from the YAML-file.

You can group fields together with a fieldset. This is highly recommended to structure your settings visually:

````
forms:
  fields:

    chapter:
      type: text
      label: chapter
      placeholder: Add Name for Chapter
      required: true

    MyFirstfieldset:
      type: fieldset
      legend: Last Modified
      fields:

        modified:
          type: checkbox
          label: Activate Last Modified
          description: Show last modified date at the end of each page?

        modifiedText:
          type: text
          label: Last Modified Text
          placeholder: Last Updated

````


The fields `modified` and `modifiedText` will be grouped in a fieldset with the legend `Last Modified`.

You can also create public forms. Learn more about forms and fields in the chapter about [forms](/forms).

