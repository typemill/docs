# renderCSS()

The Twig-function `{{ assets.renderCSS() }}` will render css-references and inline-css from plugins.

## Example Usage

It is recommended to add the `renderCSS()`-function after all your css files, and before the closing `head` tag. It is also a good practice in Twig to wrap your ressources in a block-tag.

````Twig
<head>
  <title>title</title>
  ....
  {% block stylesheets %}
    <link rel="stylesheet" href="{{ base_url }}/themes/mytheme/css/style.css">

    {{ assets.renderCSS() }}

  {% endblock %}
</head>
````



