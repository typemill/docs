#  Selectable Page Templates

If you want more flexibility for the authors, you can also provide different templates that the author can select in the author interface.

## Create a Template Selection

To create a template selection, simply add a new select field in the theme settings. In the following example, we add a new select field to the existing meta-tab. The field will be visible at the bottom of the tab. You can also define a new tab or add the select-field to an existing fieldset.

```yaml
metatabs:
  meta:
    fields:
      template:
        type: select
        label: Select a template
        options:
          standard: 'Standard'
          landingpage: 'Landingpage'      
```

## Conditionally Render a Template

In your index.twig file (or wherever you decide which template should be loaded), you can now simply add a condition to use the right template:

```twig
    {% if metatabs.meta.template == "landingpage" %}

        {% include 'landingpage.twig' %}

    {% else %}

         ...

    {% endif %}
```

## Create the Template

All you have to do now is create the template file `landingpage.twig` in your theme with the special HTML and render logic.

