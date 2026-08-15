# home

The Twig-variable `{{ home }}` indicates, if you are on the homepage or on a subpage. Returns `true` if you are on the homepage and `false` if you are on a subpage.

## Example Usage

```twig
{% block content %}

    {% if home  %}

        {% include 'home.twig' %}

    {% else %}

        {% include 'page.twig' %}

    {% endif %}

{% endblock %}
```

