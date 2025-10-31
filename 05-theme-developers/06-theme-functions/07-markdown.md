# markdown()

The Twig-function `markdown(string markdown)` takes a string with markdown syntax and returns html. It is useful if the author should be able to use markdown syntax in the theme settings, for example to create content for the footer.

## Parameter

| Parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| markdown | string | required | A sting with valid markdown syntax. |

## Return 

This function returns html-format as a string.

## Example Usage

```
<div class="sidebar-box">
    {{ markdown(settings.themes.mytheme.sidebar) }}
</div>
```

This will render the markdown-content that the author has added into the textarea of the theme-settings. The definition of the input file in the [theme-configuration](/theme-developers/configuration-file) with yaml looks like this:

````Yaml
sidebar:
  type: textarea
  label: Add some text for the sidebar with markdown
  placeholder: Add markdown here
````

