# Create a Folder Navigation

Use the Twig-variable  [item](/theme-variables/navigation) to create a sub-navigation for folders in your theme.

## List the First Content-Level of a Folder

The simple solution with all first level items of the current folder looks like this:

````
{% if item.elementType == 'folder' %}
  <ul>
    {% for sub in item.folderContent %}
      <li>{{ sub.itemName }}</li>
    {% endfor %}
  </ul>
{% endif %}
````






## List all Content-Levels of a Folder

To list all content levels within the current folder, you have to create a macro in a separate file like this:

````Twig
{% macro loop_over(folder) %}
    {% for element in folder %}
        {% if element.elementType == 'folder' %}
            <a href="{{ element.urlRel }}">{{ element.name }}</a>       
             <ul>
                 {{ macros.loop_over(element.folderContent) }}
             </ul>
            {% else %}
                <a href="{{ element.urlRel }}">{{ element.name }}</a>
            {% endif %}
        </li>
    {% endfor %}
{% endmacro %}
````




Then you have to import the macro into your template:

```Twig
{% import 'folderMacro.twig' as macros %}
{% if item.elementType == 'folder' %}
    <ul>
       {{ macros.loop_over(item.folderContent) }}
    </ul>
{% endif %}
```




## Use a Partial for the Folder Navigation

You can also create a separate partial for the folder-navigation like this:

```Twig
{% macro loop_over(folder) %}
    {% for element in folder %}
        {% if element.elementType == 'folder' %}
            <a href="{{ element.urlRel }}">{{ element.name }}</a>       
             <ul>
                 {{ macros.loop_over(element.folderContent) }}
             </ul>
            {% else %}
                <a href="{{ element.urlRel }}">{{ element.name }}</a>
            {% endif %}
        </li>
    {% endfor %}
{% endmacro %}

{% import _self as macros %}

<ul>
    {{ macros.loop_over(folder) }}
</ul>
```




And then just include the partial and pass over the folder as a variable like this:

```Twig
<nav>
    {% include 'partials/navigationFolder.twig'  with {'folder': item.folderContent} %}
</nav>
```




## List Folders Anywhere

You can also get any folder and list the content anywhere in the theme with the Twig function [getPageList](/theme-developers/theme-functions/getpagelist). See [list articles](/theme-developers/examples/list-articles) for examples.

