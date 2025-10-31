# Theme Configurations with YAML

Every theme for Typemill must have a theme configuration file. This is quickly done with a small YAML-file, which must have the same name as your theme folder. The YAML-file contains some basic informations and some optional configurations:

* **Basic Information**: Includes the theme name, description, version number, and other metadata. This section is **required**.
* **Settings**: Define default variables for your theme here. This section is **optional**.
* **Configuration Forms**: YAML definitions for theme settings that appear in the **admin panel** under system settings. This section is **optional**.
* **Configuration Tabs**: YAML definitions for page-specific theme settings. These appear in a separate **Theme** tab on each page. This section is **optional**.
* **CSP (Content Security Policy)**: A list of allowed external domains (e.g., for loading external libraries). This section is **required** if your theme loads content from other domains.

## Basic Information

The basic information in the YAML-file look like this: 

```yaml
name: My Theme Name
version: 1.0.0
description: Write what you want
author: Your name here
homepage: http://an-info-website-for-the-theme.com
license: MIT
```

The version number is critical for update checks and notifications, following a recommended schema like 1.2.3 (major.minor.bugfix). All other information will appear in the theme info boxes of the admin panel and in the theme directory.

## Add a Donation Button

You can also add a donation button for paypal like this:

```yaml
name: My Theme Name
version: 1.0.0
description: Write what you want
author: Your name here
homepage: http://an-info-website-for-the-theme.com
license: MIT
paypal: https://paypal.me/yourname
amount: 10
```

The button will appear in the theme configuration of the author panel. If your theme is uploaded to the official [theme-website](https://themes.typemill.net) of Typemill, then a donation button will appear on the overview page and on the detail page. If you want to add your theme to the theme collection, please open a new issue [on GitHub](https://github.com/typemill/typemill).

## Default Settings

Sometimes you want to use variables in your theme, for example to change the text of a button. With YAML you can easily do this: Just create a new block that starts with `settings`, and write all your settings as simple key-value-pairs. Indent them with two spaces like this: 

```
settings:
  chapter: Chapter
  start: Start
```

The settings are automatically merged with all other settings of Typemill. 

All settings are available in your themes with a simple Twig tag like this:

```
{{ settings.themes.typemill.chapter }} // prints out "Chapter".
```

Replace  `typemill` with the name of your theme like this:

````
{{ settings.themes.mytheme.chapter }}
````

## Configuration Forms

You can make your theme variables editable for the admin in the theme settings of the admin panel. To do this, just add another block that starts with `forms` and `fields`. After that, you can define a wide range of input fields with YAML. It starts with the name of the field, followed by the field definition.

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

Typemill will use these definitions and generate input fields for the admin panel on the fly, so that the admin can edit the values and customize the theme in the theme configurations. If you have defined settings with the same name as the field name (e.g. `chapter`), then the input field in the admin panel will automatically be prefilled with your settings from the YAML-file.

You can also group fields together with a fieldset. This is highly recommended to structure your settings visually:

````yaml
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

If you read the YAML definition for input fields carefully, then you will notice that the definitions are pretty similar to HTML: You simply define types and attributes like input-type, labels, and placeholders. Nearly all valid field-types and field attributes are supported.  Learn more about fields and field-types in the chapter about [forms](/forms).

## Configuration Tabs

If you want some page specific configurations like a configurable stage sections or different page templates, then you can define a tab for your theme that will appear on each content page. Just use the "metatabs" keyword and the name of your theme followed by the form configurations.

```yaml
metatabs:
  mytheme:
   fields:
    MyFirstfieldset:
      type: fieldset
      legend: Theme settings
      fields:
        stage:
          type: checkbox
          label: Stage Section
          description: Activate a stage section for this page

```

## Content Security Policy (CSP)

Typemill uses a [content security policy (csp)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) that blocks all content from external domains. If you want to include content from other domains (e.g. libraries hosted on other domains), then you have to whitelist these domains. Add a simple list of domains at the end of your configuration file.

```yaml
csp:
  - *.google.com
  - https://cdn.paddle.com
```

