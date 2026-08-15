# Create a Full Navigation

Use the Twig-variable [navigation](/theme-variables/navigation) to create a full content navigation for your theme.

## Full Navigation with All Levels

If you want to create a full navigation with all navigation levels, then you need to loop over the navigation variable recursively. You can do so in Twig with a macro.

In the following example, the macro is integrated in a separate template called "navigation.twig". You can also create a separate file with the macro (e.g. "navMacro.twig") and import it into your navigation template.

The whole usecase with the macro and the navigation in one template looks like this:

```
    {# define the macro #}
    {% macro loop_over(navigation) %}
    {% import _self as macros %}
    {% for item in navigation %}
        <li>
            {% if item.elementType == 'folder' %}
                {% if item.index %}
                    <a href="{{ item.urlRel }}">{{ item.name }}</a>
                {% else %}
                    {{ item.name }}
                {% endif %}             
                <ul>
                    {{ macros.loop_over(item.folderContent) }}
                </ul>
            {% else %}
                <a href="{{ item.urlRel }}">{{ item.name }}</a>
            {% endif %}
        </li>
    {% endfor %}
    {% endmacro %}
    {# import the macro and use it to create the navigation #}
    {% import _self as macros %}
    <nav>
        <ul class="main-menu">
            {{ macros.loop_over(navigation) }}
        </ul>
    </nav>
```






## Full Navigation with Flexible Levels

If you want to make it more flexible, you can limit the number of loops with a variable and generate a navigation to a certain navigation level. You can make this variable editable in the theme settings. The whole code might look like this:

```twig
{% set maxdepth = navidepth ? navidepth : 2 %}

{% macro loop_over(navigation, level, maxdepth) %}

    {% import _self as macros %}

    {% for element in navigation %}
        <li>
        {% if element.elementType == 'folder' and level < maxdepth %}
            <a href="{{ element.urlAbs }}">{{ element.name }}</a>       
            {% if element.contains == 'pages'  %}
                <ul class="list">
                    {{ macros.loop_over(element.folderContent,level+1, maxdepth) }}
                </ul>
            {% endif %}
        {% else %}
            <a href="{{ element.urlAbs }}">{{ element.name }}</a>
        {% endif %}
        </li>
    {% endfor %}
{% endmacro %}

{% import _self as macros %}

<ul>
    {{ macros.loop_over(navigation, 1, maxdepth) }}
</ul>
```




## Recommendation for a Theme Structure

Just as a recommendation for your theme structure: Typically you create a separate file like `navigation.twig` with all the code above. Then you place this template in a folder like `partials`. You can include this navigation-file in the `layout.twig` file, so that the navigation is included in all pages of your theme. So the structure might look like this:

```
theme
  - partials
    - layout.twig // includes navigation
    - navigation.twig
  - index.twig // extends layout.twig
```

