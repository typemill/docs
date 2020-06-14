# Activate vue.js and axios.js

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

