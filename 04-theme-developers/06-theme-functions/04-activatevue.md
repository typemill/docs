# activateVue()

The Twig-function `{{ assets.activateVue() }}` will add the vue-library to your theme.

## Example Usage

````
<body>
  ...
  {{ content }}
  ...

  {% block javascripts %}

   {{ assets.activateVue() }}

   <script src="{{ base_url }}/themes/yourtheme/js/yourVueApp.js"></script>

  {% endblock %}   
</body>
````

