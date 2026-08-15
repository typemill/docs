# logo

The Twig-variable  `{{ logo }}` holds the relative url to the uploaded logo as string. It returns `false` if no logo is uploaded in the system settings.

## Example Usage

```Twig
{% if logo %}
    <img src="{{ base_url }}/{{ logo }}" class="logo-image"/>
{% else %}
    {{ settings.title }}
{% endif %}
```

