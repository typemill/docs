# widgets

The Twig-variable `widgets` holds and array of HTML snippets that can be injected with plugins. It returns `false` if no widgets are there.

## Example Usage

All themes should implement a widgetized zone so that plugins can add html-snippets to the theme. An example is the search plugin that injects a search field as a widget. A an example for the integration of the widgets in a theme is this:

````
{% if widgets %}
    {% for index,widget in widgets %}
        <aside id="{{ index }}">
          {{ widget }}          
        </aside>
    {% endfor %}
{% endif %}
````

