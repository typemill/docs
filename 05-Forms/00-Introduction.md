# Forms with Typemill

Whenever a user wants to customize something, he will probably do it with some forms. TYPEMILL provides an easy and flexible way to generate such forms on the fly. Developers can define forms with simple YAML definitions and users (authors) can use such forms ...

* to configure a plugin,
* to configure a theme,
* to add meta-data to a page.
* to add public forms to the website (e.g. a contact form).

[TOC]

## Basics

You can define forms in the YAML configuration files of your [plugins](/plugin-developers/documentation/configuration-file) and your [theme](/theme-developers/theme-meta).

A very simple form configuration looks like this:


````
forms:
  fields:
   
    myfieldname:
      type: text
      label: Add a short text
    
    anotherfield:
      type: textarea
      label:  Add a long text
````

The rules are pretty simple:

* The form definition starts with the keyword `forms` followed by the keyword `fields`. 
* Each field definition starts with a unique field-name. 
* Below the field-name are the field definitions with the field-type, a label, attributes and more.

Typemill will use these definitions to generate forms and input fields on the fly. 

If a user fills out the form and clicks on save, then Typemill will store the input data, so that the developer has access to the data.

Let us add the example form above to the configuration-yaml file of a theme called "mytheme.yaml". Typemill will display the form in the theme configuration and store the input-data from the author in the settings-file. The developer can now output the data in the theme like this:

```
{{ settings.themes.mytheme.myfieldname }}
```

## Meta-Tabs and Public Forms

With the keyword `forms` you can generate configuration forms for your **themes** and for your **plugins**. That is great, but it is not all. Because with **plugins**, you can also create input forms for page meta-tabs or even public forms:

* If you want to add new input fields for page meta-tabs, then use the keyword `metatabs` instead of `forms`.
* If you want to create public forms like a contact form, then use the keyword `public` instead of `forms`.

Find more information and examples under metatabs and public forms.

## Validation

The field input of a user is **validated** in the backend automatically. The backend validation is pretty strict and you should always test the input fields intensively. If you run into unwanted validation errors, please report them on GitHub.

