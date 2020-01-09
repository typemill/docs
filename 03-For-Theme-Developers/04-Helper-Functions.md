# Helper Functions

With the helper functions of TYPEMILL you can create very flexible themes, you can use widgets like a search box and even list all articles of a folder.

[TOC]

## Asset Tags

Sometimes a plugin wants to add some CSS and JavaScript to your theme, for example, the Cookie Consent plugin. It adds a cookie consent popup to all pages, so that users can agree to the cookie policy of your website.

There are two Twig-tags, that allow plugins to add JavaScript and CSS. Please make sure that you add these tags to your theme, because otherwise, the plugins won't work:

- `{{ assets.renderCSS() }}`
- `{{ assets.renderJS() }}`

It is recommended to add the CSS tag after all your css files, and before the closing `<head>` tag. It is also a good practice in Twig to wrap your ressources in a block-tag. You can read more about this in the Twig chapter.

````
<html>
 <head>
  <title>title</title>
  ....
  ....
  {% block stylesheets %}
    <link rel="stylesheet" href="{{ base_url }}/themes/mytheme/css/style.css" />
		
    {{ assets.renderCSS() }}
			
  {% endblock %}
 </head>
````

The same for JavaScript: It is recommended to place all JavaScript at the end of the page before the closing `<body>` tag. And you should wrap all your JavaScript in a block-element, too:

````
<body>
  ...
  {{ content }}
  ...
  ...
  
  {% block javascripts %}
		
   <script src="{{ base_url }}/themes/yourtheme/js/my.js"></script>
   
   {{ assets.renderJS() }}
		
 {% endblock %}		
</body>
````

## Activate Tachyons

Typemill uses the CSS library [Tachyons](https://tachyons.io/). If you want to use the library for your theme, then you can activate it in the header like this:

````
<html>
 <head>
  <title>title</title>
  ....
  ....
  {% block stylesheets %}
    <link rel="stylesheet" href="{{ base_url }}/themes/mytheme/css/style.css" />

    {{ assets.activateTachyons() }}
    
    {{ assets.renderCSS() }}
      
  {% endblock %}
 </head>
````

## Activate Vue.js and Axios

Typemill uses the frontend-library Vue.js and Axios.js for http-requests. You can activate both for your theme like this:

````
<body>
  ...
  {{ content }}
  ...
  ...
  
  {% block javascripts %}
    
   <script src="{{ base_url }}/themes/yourtheme/js/my.js"></script>
   
   {{ assets.activateAxios() }}
   {{ assets.activateVue() }}

   {{ assets.renderJS() }}
    
 {% endblock %}   
</body>
````

Axios will be instanciated with the variable `myaxios` with the base-url that is needed for axios.

## List Articles

Typemill is not a blog engine, but it provides an easy way to loop through all articles of a folder and to display the articles as a list. To do so, just use the `{{ item }}` variable of the folder and loop through each sub-item like this:

```
    <ul>
      
      {% for article in item.folderContent %}

        <li>
          <a href="{{ article.urlAbs }}"><h2>{{ article.name }}</h2></a>
        </li>
          
      {% endfor %}
    </ul>

```

The example above is pretty limited because the item array does only provide the name of the article and no other informations like the author or the description. To get those informations we can use the helper function `{{ getPageMeta(settings,pageItem) }}` and fetch the meta-information for each article:

```
    <ul>
      
      {% for article in item.folderContent %}

        {% set page = getPageMeta(settings, article) %}

        <li>
          <a href="{{ article.urlAbs }}">
              <h2>{{ article.name }}</h2>
              <small>{{ page.meta.modified }} | {{ page.meta.author }}</small>
              <p>{{ page.meta.description }}</p>
          </a>
        </li>
          
      {% endfor %}
    </ul>

```

## Render Markdown

Let's say you want to create a sidebar box in your theme, and the user should be able to add text and use Markdown to format it, e.g. add a link. You can realize this with TYPEMILL's `{{ markdown() }}` helper function in two steps:

In the first step you define a textarea for the sidebar widget in the YAML-file of your theme like this:

````
        sidebar:
          type: textarea
          label: Add some text for the sidebar with markdown
          placeholder: Add markdown here
````

Then you can render out the Markdown content in your theme like this:

````
<div class="sidebar-box">
    {{ markdown(settings.myplugin.sidebar) }}
</div>
````

## Render Widgets

Themes should implement a widgetized zone like this somewhere:

````
{% if widgets %}
    {% for index,widget in widgets %}
        <aside id="{{ index }}">
      {{ widget }}          
  </aside>
    {% endfor %}
{% endif %}
````

Widgets are simply some HTML snippets that can be injected by plugins. A good example for a widget is a Search field, that is added with the existing Search plugin.

Keep reading the documentation to learn all the details. 