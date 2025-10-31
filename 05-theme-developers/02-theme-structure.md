# Theme Structure

Typemill does not predefine a certain structure for a theme. It just requires some mandatory files. Everything else is up to you.

## Minimal Theme Structure

For a minimal theme, Typemill requires only a small set of mandatory files:

````
/mytheme
- index.twig
- 404.twig
- thumbnail.png
- mytheme.png
- mytheme.yaml
````

## Naming Rules

Use a unique, short, and descriptive name for your theme with alpha characters [a-z].

- Use the theme name for your folder name.
- Use the theme name for the preview image.
- Use the theme name for the YAML configuration file.

## Short Explanation of Mandatory Files

* `index.twig`: This template will render all content files. You can extend the index-file with other files.
* `404.twig`: This is the template for a 404 error message.
* `mytheme.yaml`: This is the configuration file with the version, the author name, license, and other information about your theme.
* `mytheme.png`: This is a preview picture of your theme with a minimum width of 800px.
* `thumbnail.png`: This is a preview picture of your theme for the admin area.

## Extended Theme Structure

In Twig, you can include and extend templates, and create a template hierarchy. With these features, you can create a more complex theme with a structure. Read more about it in the [chapter about Twig](/theme-developers/twig).

If you don't have an idea how to start, then you can follow the recommendation below.

```
- /css
    - /style.css
    - /another.css
- /js
    - /javascript.js
- /img
    - /icon.png
    - /favicon.ico
    - /themeLogo.jpg
- /pages
    - /blog.twig
    - /file.twig
    - /folder.twig
    - /glossary.twig
    - /home.twig
- /partials
    - /breadcrumb.twig
    - /navigation.twig
    - /paging.twig 
    - /header.twig
    - /footer.twig
- /index.twig
- /layout.twig
- /404.twig
- /thumbnail.png
- /themeName.png
- /themeName.yaml
```

