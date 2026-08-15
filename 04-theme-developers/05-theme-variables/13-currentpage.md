# currentpage

The Twig-variable `currentpage` holds the number of the current page from an url with a valid pagination like `/p/2`. It returns the page number as `int` or `false` if there is no valid pagination.

A pagination is valid with these conditions:

* An url-structure with segment `/p/` followed by an int-number.
* The item must be a folder.
* The folder must contain posts.
* The page number must map to an existing page.

## Example Usage

You can check the code of the cyanine theme to understand how a pagination works:

````
{% if item.elementType == 'folder' and item.contains == 'pages' %}

{% set pagesize = 10 %}
{% set pages = ( item.folderContent|length / pagesize)|round(0, 'ceil') %}
{% set currentpage = currentpage ? currentpage : 1 %}
{% set currentposts = (currentpage - 1) * pagesize %}
<ul class="post list pa0">

    {% for element in item.folderContent|slice(currentposts, pagesize) %}

        {% set post = getPageMeta(settings, element) %}
        {% set date = element.order[0:4] ~ '-' ~ element.order[4:2] ~ '-' ~ element.order[6:2] %}

        <li class="post-entry">
            <header>
                <a class="link f-link underline-hover" href="{{ element.urlAbs }}"><h2 class="mt4 mb2">{{ post.meta.title }}</h2></a>
                <div class="mt3"><small><time datetime="{{date}}">{{ date | date("d.m.Y") }}</time> | {{ post.meta.author }}</small></div>
            </header>
            <p>{{ post.meta.description }}</p>
        </li>

    {% endfor %}

    {% if pages > 1 %}
        <hr class="mv4">
        <p>Page: 
            {% for i in 1 .. pages %}
                {% if i == currentpage %}
                    {{i}}
                {% else %}
                    <a class="page" href="{{ item.urlAbs }}/p/{{i}}">{{i}}</a>
                {% endif %}
            {% endfor %}
        </p>
    {% endif %}

</ul>

{% endif %}
````

