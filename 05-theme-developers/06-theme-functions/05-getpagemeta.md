# getPageMeta()

The Twig-function `{{ getPageMeta(settings,pageItem) }}` returns the metadata of a page as an array. The function can be used to list meta-information like meta-title and meta-description of pages inside a folder.

## Parameter

| Parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| settings | array | required | The [settings](/theme-developers/theme-variables/settings) variable.  |
| pageItem | array | required | The [item](/theme-developers/theme-variables/item) variable of a page. |

## Return 

This function return the meta-data of a page as an array.

## Example Usage

If the current [item](/theme-developers/theme-variables/item) is a folder, then you can loop through `item.folderContent`. Alternatively you can get items from the [navigation](/theme-developers/theme-variables/navigation) with [getPageList](/theme-developers/theme-functions/getPageList). 

```
    <ul>

      {% for article in item.folderContent %}
        {% set page = getPageMeta(settings, article) %}
        <li>
          <a href="{{ article.urlAbs }}">
              <h2>{{ article.name }}</h2>
              <small>{{ page.meta.modified }} | {{ page.meta.author }}</small>
              <p>{{ page.meta.description }}</p>
          </a>
        </li>

      {% endfor %}
    </ul>
```

