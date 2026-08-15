# renderJS()

The Twig-function `{{ assets.renderJS() }}` will render JavaScript-references and inline JavaScript from plugins.

## Example Usage

It is recommended to place all JavaScript at the end of the page before the closing `body` tag. You should wrap all your JavaScript in a block-element.

````Twig
<body>
  ...
  {{ content }}
  ...

  {% block javascripts %}

   <script src="{{ base_url }}/themes/yourtheme/js/my.js"></script>

   {{ assets.renderJS() }}

 {% endblock %}

</body>
````



