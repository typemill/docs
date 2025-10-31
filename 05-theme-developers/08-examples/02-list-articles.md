# List Articles

Typemill provides an easy way to loop through all articles of a folder and to display the articles as a list. This is useful if you want to create a list of articles from a news- or blog-section with the content type `post` or from any content-folder with the content-type `pages`.

## List Articles with a Folder-Item

In the most simple case you can just use the [item-variable](/theme-variables/item) of a `folder` and loop through the `folderContent` like this:

```
    <ul>

      {% for article in item.folderContent %}
        <li>
          <a href="{{ article.urlAbs }}"><h2>{{ article.name }}</h2></a>
        </li>

      {% endfor %}
    </ul>
```





The example above is pretty limited because the item array does only provide the name of the article and no other informations like the author or the description. These informations are typically part of the meta-information which are stored in the yaml-file of a page.

## List Articles with Meta-Information

To get the meta-informations of each page inside of a folder, you can use the helper function [getPageMeta](/theme-functions/getpagemeta) like this:

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





## List Articles Anywhere

If you want to list articles, then you are not limited to the item of the current page. You can also list articles from any folder anywhere, even on the startpage. this way you can create some kind of **blog-layout**.

To get a list of articles from a folder, use the function [getPageList](/theme-developers/theme-functions/getpagelist) and inject the full navigation, the folder that should be listed and the base-url. You can also create an input field in your theme-settings, so that the author can add a content folder for articles.

The following snippet from a theme demonstrates the logic:

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

