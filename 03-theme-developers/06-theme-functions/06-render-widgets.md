# Render Widgets

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

Widgets are simply some HTML snippets that can be injected with plugins. A good example for a widget is a search field, that is added with the existing search plugin.

