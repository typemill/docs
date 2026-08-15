# home_url

The Twig-variable `{{ home_url }}` holds the base url of your website and optionally the segment of the current project. Use it to create a home link to the general homepage or to the current project homepage.

## Example Usage

The home url is especially useful for a home-link that is typically used for a logo. 

````
<a href="{{ home_url }}">
   {% if logo and theme.logoimg %}
      <img src="{{ base_url }}/{{ logo }}" alt="Logo">
   {% endif %}
   {% if theme.logotext %}
       <span>{{ settings.title }}</span>
   {% endif %}
</a>
````

The administrator can change the behavior of the home url in the project tab of the system settings with the checkbox "[Project Homepage](/admin-guide/project-tab)".

