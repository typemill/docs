# Field Overview

Fields and forms for user input are defined in YAML. YAML is a simple configuration-syntax that transforms into an object or an array. The forms can be used as [public forms](/for-plugin-developers/documentation/public-forms) in the frontend (e.g. for a contact form), for plugin settings, and for theme settings. TYPEMILL will read all your field definitions, and create a user interface on the fly.

[TOC]

## Basics

The fields are defined in the [YAML configuration file](/for-plugin-developers/documentation/configuration-file) of a plugin or theme. Fields for plugins or themes start with the keywords `forms` and `fields`, followed by a list of fields like this:

````
forms:
  fields:
   
    myfieldname:
      type: text
      label: My Field
    
    anotherfield:
      type: text
      label: Another Field
````

Public forms are defined in the same way but, start with the keywords `public` and `fields`:

````
public:
  fields:
   
    myfieldname:
      type: text
      label: My Field
    
    anotherfield:
      type: text
      label: Another Field
````

The field input of a user is **validated** in the backend automatically. The backend validation is pretty tight, so that you should always test the input fields intensively. If you run into unwanted validation errors, please report them on GitHub.

Typically a field definition consists of:

* The **field-type** (text, select, checkbox and more).
* A **label**.
* A **description** (displayed below the field).
* A **help-text** (displayed as question mark that displays an info box on hover).
* **Attributes** like "required", "placeholder", and more.

## Field Types and Attributes

Right now, TYPEMILL provides the following field types:

````
type: checkbox
type: checkboxlist
type: color
type: date
type: email
type: hidden
type: number
type: password
type: radio
type: select
type: text
type: textarea
type: tel
type: url
type: fieldset
type: paragraph
````

TYPEMILL supports these boolean attributes (with value: true):

````
auto: true
checked: true
disabled: true
formnovalidate: true
multiple: true
readonly: true
required: true
````

**Very important**: If you use **required: true**, then you have to add a default value for the field in the [settings definitions](/for-plugin-developers/documentation/configuration-file) of your YAML-file. Otherwise, the user cannot store the plugin settings due to frontend validation.

TYPEMILL supports the following attributes with values:

````
id: 'myId'
class: 'myClass'
pattern: '[0-9]{4}'
placeholder: 'my placeholder-text here'
min: 1
max: 10
size: 5
rows: 5
cols: 5 
````

The `pattern` attribute accepts every valid regex for input validation in the frontend. Please note, that there is also a backend validation that might conflict with your frontend validation. So please double check your validation pattern, and test the input intensively. You should, of course, only use attributes like `required`, `placeholder`, `rows`, or `cols` if these attributes are valid in HTML for the field type.

## Examples

Check out the following examples. These are only for demonstration, and you can combine the attributes in all ways that are valid for HTML forms.

### checkbox

Creates a simple checkbox. Note that the HTML label that is on the right side of the checkbox is created with `checkboxlabel`. The `label` is added above the checkbox so that it is looks like the label of the other field-types.

````
forms:
  fields:
   
    mycheckbox:
      type: checkbox
      label: Label (Headline) above the checkbox
      checkboxlabel: HTML-label on the right side of the checkbox
      checked: true
````

### checkboxlist

If you want a collection of several checkboxes grouped together, then you can use a checkboxlist like this:

````
forms:
  fields:
   
    mylistofcheckboxes:
      type: checkboxlist
      label: Please check all checkboxes
      options:
        checkbox1: Label for checkbox 1
        checkbox2: Label for checkbox 2
        checkbox3: Label for checkbox 3
````

### color

Adds a color picker. If you use this for a frontend form, then the standard HTML5 color picker will be used. You have to style it yourself, if you don`t like the standard. For theme settings and plugin settings, an individual color picker is used.

````
forms:
  fields:
   
    mycolors:
      type: color
      label: Background-Color
      placeholder: Add hex color value like #ffffff
      help: If you do not know the hex-values, then google it.
      description: This will change the background color of your page.
      required: true
````

### date

The standard HTML5 datepicker will be used.

````
forms:
  fields:
   
    mydate:
      type: date
      label: Please choose a day.
````

### email

Will only accept valid e-mail-formats.

````
forms:
  fields:
   
    myemail:
      type: email
      label: Add your e-mail
      placeholder: your@email-adress.io
````

### fieldset

If you have a lot of fields, then you can group them together with a fieldset. You can use any fields inside a fieldset of course.

````
forms:
  fields:
   
    myfieldset:
      type: fieldset
      legend: Appears as legend or headline for the field-group
      fields: 
        mytextfield:
          type: text
          label: My Label
        mycheckbox:
          type: checkbox
          label: Please check
          checkboxlabel: Do you want to check me?
````

### hidden

If you really need a hidden field, don't forget to add a value.

````
forms:
  fields:
   
    myhiddenfield:
      type: hidden
      value: Do not forget to add a value
````

### number

Accepts only numbers.

````
forms:
  fields:
   
    mynumber:
      type: number
      label: Add a number
      min: 1
      max: 10
````

### password

There is probably no use case for this right now.

````
forms:
  fields:
   
    mypassword:
      type: password
      label: Enter your password
````

### radio

Radio buttons are created with options like this:

````
forms:
  fields:
   
    myradiobuttons:
      type: radio
      label: Please Choose
      options:
        option1: Yes
        option2: No
        option3: I do not know
````

### select

Select fields are created with options like this:

````
forms:
  fields:
   
    theme:
      type: select
      label: Select A Theme
      options:
        edgeless: Edgeless Theme
        block: Block Theme
        classic: Classic Theme
````

### text

Generates a standard text-input field.

````
forms:
  fields:
   
    mytextfield:
      type: text
      label: Please add a title
      placeholder: My Title
      size: 50
      pattern: [a-zA-Z0-9 ]
      class: 'standard-title'
      id: 'mytitle'
````

### textarea

A textarea does not accept any HTML tags or code, but it always accepts Markdown. If you render textarea input as Markdown, you should notify the user about this (use the label or the description for this).

````
forms:
  fields:
   
    myfieldname:
      type: textarea
      label: Add a message
      rows: 5
      description: You can use markdown here.
      help: Check the website XYZ for a markdown reference
````

### tel

Gernerates a HTML5 input field for telephone numbers.

````
forms:
  fields:
   
    mytelfield:
      type: tel
      label: Add your phone number
````

### url

This field only accepts a full URL with a protocol like http, https, ftp, or similar.

````
forms:
  fields:
   
    myurlfield:
      type: url
      label: Please add a valid url
      placeholder: https://typemill.net
````

### paragraph

A paragraph is a pure text. A paragraph field accepts Markdown, but does not provide any other options than a value.

````
forms:
  fields:
   
    legalnotice:
      type: paragraph
      value: This is my text that should be displayed in the form.
````

Paragraphs can be useful for [public forms](/for-developers/documentation/public-forms), e.g. if you want to add a legal notice or a longer explanation. It is also possible to use the user input of another plugin-field for the paragraph, so the user can edit, for another example, the legal notice of a contact form.

## Example for a complete yaml configuration

To sum it up, this is a complete example of a YAML configuration file for a plugin with the meta description, a default value, and a field definition for user input:

````
name: Example Plugin
version: 1.0.0
description: Add a short description
author: Firstname Lastname
homepage: http://your-website.net
licence: MIT
settings:
  theme: 'edgeless'
forms:
  fields:
    theme:
      type: select
      label: Select a Theme
      required: true
      options:
        edgeless: Edgeless
        block: Block
        classic: Classic
````

