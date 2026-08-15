# breadcrumb

The Twig-variable `{{ breadcrumb }}` holds the breadcrumb navigation for the page. It is a simple one dimensional array of item objects.

## Example Usage

Loop over the breadcrumb and print the elements out:

    <ul class="breadcrumb">
    {% for element in breadcrumb %}
        <li><a href="{{ element.urlRel }}">{{ element.name }}</a></li>
    {% endfor %}
    </ul>

For each item in the breadcrumb, all information of the [item-object](/theme-developers/theme-variables/item) is available.

