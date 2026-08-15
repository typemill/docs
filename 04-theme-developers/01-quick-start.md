# Quick Start for Theme-Developers

Are you a professional web developer, and you don't want to read the whole documentation? No problem, this is all you need to know to create your own theme for Typemill.

!! Please use the new developer kit for theme development. It comes with a full Typemill installation and a **complete guide** that walks you through the new **Dev Theme**. You can download or clone it from [GitHub](https://github.com/typemill/dev).

## Theme Folder

All themes are located in the `theme` folder of Typemill. You can add a new folder for your theme there. The name of your folder is the name of your theme.

## Change Theme

You can choose the theme in author panel in Typemill.

## Theme Structure

There are only four files that are required for a minimal theme:

* `index.twig`: All content files will be rendered with this template. 
* `404.twig`: This is the template for a 404 error message.
* `themeName.yaml`: A configuration file with the version, the author name, license, and other information about your theme.
* `themeName.jpg`: A preview picture of your theme with a minimum width of 800px;

Additionally to these standard files you can structure your theme like you want. Read more about the [theme structure here](/theme-developers/theme-structure).

## Theme-YAML

The `themeName.yaml` must have the same name as your theme folder. The yaml-file starts with some general information about the theme:

````
name: My Theme Name
version: 1.0.0
description: Write what you want
author: Your name here
homepage: http://an-info-website-for-the-theme.com
license: MIT
````

You can also add settings for your themes in the yaml-file like this:

````
settings:
  chapter: Chapter
  start: Start
````

The settings are automatically merged with all other Typemill settings, and are available on all pages, so you can access your theme variables like this:

````
{{ settings.themes.typemill.chapter }} // prints out "Chapter".
````

Finally you can make your theme variables editable for the user in the author panel. Just add a form definition in your yaml like this:

````
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
````

This will create input forms in the author panel. The input forms will be prefilled with the settings-values of your yaml-file. Read all about form-definitions in the [chapter about forms](/forms).

If you want to include libraries or other resources from other domains, then you have to whitelist those domains for the content security policy. Just add it at the end of your yaml-file like this:

```
csp:
  - *.google.com
  - https://cdn.paddle.com
```

Read more in the chapter about the [yaml-configuration-file](/theme-developers/configuration-file).

## Twig

Typemill uses Twig as a template language. You are probably familiar with it. If not: Twig is a widespread template language that is very easy to learn. It is shorter and safer to use than pure PHP. If you are not familiar with Twig, please check the [Twig documentation](https://twig.symfony.com/) from Symfony Framework.

## Theme Variables

Typemill provides theme variables to fill your templates with dynamic content. Read all about it in the chapter about [theme variables](/theme-developers/theme-variables).

* `home`: True if it is the homepage, false if not.
* `title`: The title of the page as string (first h1 headline from content page).
* `content`: HTML content transformed from the markdown file.
* `metatabs`: All meta-data from the yaml-file of a page. Use metatitle, metadescription, and other meta-infos from this variable.
* `navigation`: Multidimensional array of item-objects. Each item-object represents a file or a folder. You can use this variable to create a navigation with a Twig-macro. A macro in Twig is the same as a recursive function in PHP. 
* `item`: An object of the actual page. It contains all the details like the name, the url, the path, the chapter, as well as the next and the previous items for a pagination.
* `breadcrumb`: One dimensional array. It contains the breadcrumb navigation of the page. Use a loop like  `{% for element in breadcrumb %}` to print it out.
* `settings`: In this variable, you will find all the settings like the navigation-title, the author, the theme, the theme variables, or the copyright.
* `base_url`: The base-url of the application. Useful for a link to the homepage or to generate absolute references.
* `logo`: If set, the relative link to the logo in the media library.
*  `favicon`: If set, an array with relative links to the favicons in the media library.
* `currentpage`: If you have a paging for posts in the url like /p/23, then the currentpage holds the page number, otherwise false.

You can print out each variable with the twig-tag `{{ dump(navigation) }}` and inspect the content. This is probably the easiest way to familiarize yourself with the possibilities for themes.

## Theme Functions and Assets

Typemill provides some useful theme functions. You can render assets from plugins, acitvate vue and axios, list articles from folders, manipulate images, render markdown, and render widgets from plugins. Read all about it in the chapter about [theme functions](/theme-developers/theme-functions).

Plugin developers need a way to add their own CSS and JavaScript to your theme. This is why you should always include the following functions into your theme:

* `{{ assets.renderCSS() }}`: Put this before the closing `</head>` tag of your theme.
* `{{ assets.renderJS() }}`: Put this before the closing `</body>` tag of your theme. 

Some plugins also need certain html-elements, for example an input field for a search. Plugins can add these elements with the widget-feature. Always [render the widgets](l/theme-developers/theme-variables/widgets) somewhere in your theme, otherwise some plugins won't work.

## Content-Styling

If you create a theme, make sure that all content types (headlines, paragraphs, tables) are styled properly. You can use the [Markdown Test Page](/info/markdown-test) to check the styling of all content elements.

## Read more

If this page hasn't answered all of your questions, then please read the full developer manual. In less than one hour, you can develop your own themes for Typemill like a pro.

Happy coding!

