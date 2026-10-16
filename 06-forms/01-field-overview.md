# Field Overview

You need a checkbox, a date and a textarea? No problem, Typemill provides it all...

## Elements of a Field

Typically a field definition consists of:

* The **field-type** (text, select, checkbox and more).
* A **label**.
* A **description** (displayed below the field).
* A **help-text** (displayed as question mark that displays an info box on hover; only rendered in public forms).
* **Attributes** (like "required", "placeholder", and more).

## Field Types

Right now, Typemill provides the following field types:

````
type: checkbox
type: checkboxlist
type: color
type: date
type: email
type: hidden
type: image
type: file
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
type: customfields (only meta-tabs)
type: codearea (only meta-tabs)
````

Configuration forms for themes and plugins as well as the forms in meta-tabs are rendered with Vue.js, while public forms are rendered with PHP. Not every field type works in every context:

* `customfields` and `codearea` are only available in meta-tabs.
* `image` and `file` work in meta-tabs and in the configuration forms of themes and plugins, but not in public forms.
* All other field types work in all contexts.

## Field Attributes

Typemill supports these boolean attributes (with value: true):

````
autofocus: true
checked: true
disabled: true
formnovalidate: true
multiple: true
readonly: true
required: true
````

Typemill supports the following attributes with values:

````
id: 'myId'
css: 'myClass'
autocomplete: 'new-password'
pattern: '[0-9]{4}'
placeholder: 'my placeholder-text here'
maxlength: 60
min: 1
max: 10
size: 5
rows: 5
cols: 5 
````

Use `css` (not `class`) to add your own css-classes to the field wrapper. In meta-tabs you can also use the css-class `w-half` to display two fields side by side.

The `pattern` attribute accepts every valid regex and is always used for the **backend validation**. In public forms it is also rendered as a HTML attribute for the frontend validation. Please note that the backend validation might conflict with your frontend validation pattern, so please double check your validation pattern and test the input intensively. You should, of course, only use attributes like `required`, `placeholder`, `rows`, or `cols` if these attributes are valid in HTML for the field type.

## Backend Validation

All user input is validated on the server side. The attributes `required`, `maxlength`, `min`, `max` and `pattern` are validated for all field types. Additionally, each field type has its own validation rules:

| Field type | Backend validation | 
|:---|:---|
| checkbox | If `required` is set, the checkbox must be checked. | 
| checkboxlist | Each value must exist in the defined `options`. | 
| codearea | Maximum 50000 characters, HTML is allowed. | 
| color | Hex-color format like `#fff` or `#ffffff`. | 
| customfields | Array of key-value pairs, no empty keys or values, no HTML. Keys must match `keypattern` (default `[a-z0-9]`), values must match `valuepattern` if defined. | 
| date | Valid date format. | 
| email | Valid e-mail format. | 
| image | No HTML, maximum 1000 characters, valid image file types. | 
| file | No HTML, maximum 1000 characters. | 
| number | Integer numbers only, no floats. | 
| paragraph | No HTML, maximum 10000 characters. | 
| password | Maximum 500 characters. | 
| radio | Value must exist in the defined `options`. | 
| select | Value must exist in the defined `options`. | 
| text | No HTML, maximum 1000 characters. | 
| textarea | No HTML, maximum 10000 characters. Markdown is allowed. | 
| url | Valid URL with protocol, maximum 200 characters. | 

Field types without a dedicated rule (for example `tel` or `hidden`) fall back to a very strict default validation that only accepts letters, numbers, spaces, underscores and hyphens (`/^[\pL0-9_ \-]*$/u`) with a maximum of 1000 characters. A phone number like `+49 (0) 123` would **not** pass this validation, so test fields like `tel` carefully.

## Examples

Just copy & paste the required field-types from the examples below to create your desired form. You can combine the attributes in all ways that are valid for HTML forms.

### checkbox

Creates a simple checkbox. Note that the HTML label (which appears on the right side of the checkbox) is created with `checkboxlabel`. The `label` is added above the checkbox so that it is looks like the label of the other field-types.

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

Adds a color picker. All forms use the standard HTML5 color picker. The backend validation only accepts hex color values like `#fff` or `#ffffff`.

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

### customfields

Adds repeatable fields for keys and values, similar to the customfields of WordPress. 

````
metatabs:
    mytab:
        fields:
            myfield:
                type: customfields
                label: Example for customfields
                description: Please add keys and values.
````

User input from custom fields will be stored like this in the page yaml:

````
mytab:
    myfield:
        userkey: userinput for value
        anotheruserkey: more userinput for value
````

You can use the content of customfields in a template like this: 

````
{{ metatabs.mytab.myfield.userkey }} # will output "userinput for value"
````

By default the customfields are designed with the key left and a small textarea on the right side. If you want to utilize the customfields for larger text blocks, then you can use a variant with the key on top and a larger textarea below with the css class `cf-large`:

```yaml
metatabs:
    mytab:
        fields:
            myfield:
                type: customfields
                label: Example for customfields
                description: Please add keys and values.
                css: 'cf-large'
```

You can customize the validation of both fields. By default, the left field (key) will have a strict validation with `[a-z0-9]`, the right field only uses strip-tags to avoid any kind of code there. You can add individual regex-patterns to both with `keypattern` and `valuepattern`. Please do not add any delimiters to the patterns, they will be added by the code. An example:

````
metatabs:
    mytab:
        fields:
            myfield:
                type: customfields
                label: Example for customfields
                description: Please add keys and values.
                keypattern: '[a-zA-Z0-9_ -]+'
                valuepattern: '[a-z0-9]+'
````

This pattern is less strict and allows some special characters like "_ -" for keys.

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

If you have a lot of fields, then you can group them together with a fieldset. You can use any fields inside a fieldset of course. Fieldsets work in all contexts, including meta-tabs.

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

If you really need a hidden field, don't forget to add a value. The way you set the value depends on the context:

* In meta-tabs and in configuration forms for themes and plugins (rendered with Vue.js), add the value directly to the field definition:

````yaml
forms:
  fields:
    myhiddenfield:
      type: hidden
      value: Do not forget to add a value
````

* In public forms (rendered with PHP), add the value to the `settings` block of your plugin configuration file:

````yaml
public:
  fields:
    myhiddenfield:
      type: hidden
settings:
  myhiddenfield: Do not forget to add a value
````

Alternatively you can hide a visible field with the css-class `hidden` like this: `css: 'hidden'`. This is useful if you want to store a value that is generated automatically, for example a page id.

### image

You can add images in meta-tabs, themes and plugins. Images do not work with public forms right now and they do not support any additional attributes.

````yaml
forms:
  fields:
    myimagefield:
      type: image
      description: Maximum size of images is 2 MB. Only accepts png, jpeg and gif.
````

### file

You can add files in meta-tabs, themes and plugins. Files do not work with public forms right now and they do not support any additional attributes.

```yaml
forms:
  fields:
    myfilefield:
      type: file
      description: Maximum size of files is X MB.
```

### number

Accepts only integer numbers (no floats), the backend validation will reject decimal numbers.

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

If you want to hide information on user input. The password generator button is only available in forms that are rendered with Vue.js (meta-tabs and configuration forms for themes and plugins), not in public forms.

````
forms:
  fields:
    mypassword:
      type: password
      label: Enter your password
      autocomplete: new-password # prevent browsers to autofill and update
      generator: true # add a button to generate passwords
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

If you want the user to select a user role, then you can populate the options automatically with all existing user roles. Simply add `dataset: userroles` to the field definition (works in meta-tabs and in the configuration forms of themes and plugins):

````
forms:
  fields:
    allowedrole:
      type: select
      label: Minimum user-role
      dataset: userroles
      options:
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
      maxlength: 50
      pattern: '[a-zA-Z0-9 ]'
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

Generates a HTML5 input field for telephone numbers. Note that the backend validation for this field type is very strict and only accepts letters, numbers, spaces, underscores and hyphens (see the [backend validation](#backend-validation) above), so characters like `+` or brackets will not pass the validation.

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

A paragraph is a pure text that you can use to add explanations or legal notices to your form. A paragraph accepts Markdown in public forms.

````
forms:
  fields:
    legalnotice:
      type: paragraph
      value: This is my text. # for public forms
      label: Legal notice # for forms in meta-tabs, themes and plugins
      description: This is my text. # for forms in meta-tabs, themes and plugins
````

In **public forms** the content of the paragraph is rendered from the `value` attribute. In forms that are rendered with Vue.js (meta-tabs and configuration forms for themes and plugins) the `value` attribute is ignored and the paragraph renders the `label` and the `description` instead. So if you want to support both contexts, add all three attributes.

Paragraphs are useful for [public forms](/forms/public-forms), e.g. if you want to add a legal notice or a longer explanation. It is also possible to use the user input of another plugin-field for the paragraph, so the admin can edit, for example, the legal notice of a contact form in the plugin settings.

