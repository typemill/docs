# Tachyons CSS Library

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

