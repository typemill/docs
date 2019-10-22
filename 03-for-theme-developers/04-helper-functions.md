# Helper Functions

TYPEMILL provides some useful helper Twig helper functions for your theme: 

* A **base url** that returns the root url
* The **asset tags** allow plugins to inject css and js-ressources or inline-code.
* A **widget tag** allow plugins to inject html-snippets like a search field.
* A **markdown render function** to render additional markdown content in your theme.

## Base URL

Whenever you want to create some urls in your theme, you can build them with the base_url tag. The base url always returns the basic url to your application. Usually this is the domain-name, but if you develop on localhost, it can also be something like `http://localhost/your-project-folder`.

````
{{ base_url }}
````

If you develop your theme, the base url is pretty useful if you want to include some assets like CSS or JavaScript. You can reference to these files like this: 

````
<link rel="stylesheet" href="{{ base_url }}/themes/typemill/css/style.css" />
````

## Asset Tags

Sometimes a plugin wants to add some CSS and JavaScript to your theme. For example, the cookieconsent-plugin. It adds a cookie-consent popup to all pages, so that users can agree to the cookie policy of your website.

There are two Twig-tags, that allow plugins to add JavaScript and CSS. Please make sure that you add these tags to your theme, because otherwise, the plugins won't work:

- `{{ assets.renderCSS() }}`
- `{{ assets.renderJS() }}`

It is recommended to add the CSS tag after all your css-files and before the closing head-tag. It is also a good practice in Twig, to wrap your ressources in a block-tag. You can read more about this in the Twig-chapter.

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

The same for JavaScript: It is recommended to place all JavaScript at the end of the page before the closing body-tag. And you should wrap all your JavaScript in a block-element, too:

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

## Widgets

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

Widgets are simply some html-snippets, that can be injected by plugins. A good example for a widget is a search-field, that is added with the existing search-plugin.

## Render Markdown

Let's say you want to create a sidebar box in your theme and the user should be able to add text and use markdown to format it, e.g. add a link. You can realize this with the `{{ mardown() }}`-helper-function of TYPEMILL in two steps:

In the first step you define a textarea for the sidebar widget in the YAML-file of your theme like this:

````
        sidebar:
          type: textarea
          label: Add some text for the sidebar with markdown
          placeholder: Add markdown here
````

Then you can render out the markdown content in your theme like this:

````
<div class="sidebar-box">
    {{ markdown(settings.myplugin.sidebar) }}
</div>
````

Proceed with the documentation to learn all the details. 
