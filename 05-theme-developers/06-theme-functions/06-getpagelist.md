# getPageList()

The Twig-function `getPageList(navigation, urlRel, baseurl)` returns the item of a folder that is referenced with a relative url. The function is useful if you want to list articles from any folder on any page, for example the articles of a news-folder on the startpage.

## Parameter

| Parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| navigation | array | required | The full [navigation](/theme-developers/theme-variables/navigation).  |
| relUrl | string | required | Relative url to a folder-page, can be a valid string from the theme settings or the UrlRel-property from an [item](/theme-developers/theme-variables/item). |
| baseurl | string | required | The [baseurl](/theme-developers/theme-variables/baseurl). |

## Return 

This function returns the item of a folder.

## Example Usage

```
  {% set pagelist = getPageList(navigation, settings.themes.emergency.listpages, base_url) %}
  <ol class="postlist" reversed>
    {% for element in pagelist.folderContent %}
      {% set post = getPageMeta(settings, element) %}
      {% set date = element.order[0:4] ~ '-' ~ element.order[4:2] ~ '-' ~ element.order[6:2] %}
      <li>
         <a href="{{ element.urlAbs }}">{{ post.meta.title }}</a> <time datetime="{{date}}">({{ date | date("d.m.Y") }})</time>
      </li>

    {% endfor %}
  </ol>
```

