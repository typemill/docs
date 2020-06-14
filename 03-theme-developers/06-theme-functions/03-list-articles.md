# List Articles

Typemill is not a blog engine, but it provides an easy way to loop through all articles of a folder and to display the articles as a list. To do so, just use the `{{ item }}` variable of the folder and loop through each sub-item like this:

```
    <ul>
      
      {% for article in item.folderContent %}
        <li>
          <a href="{{ article.urlAbs }}"><h2>{{ article.name }}</h2></a>
        </li>
          
      {% endfor %}
    </ul>
```

The example above is pretty limited because the item array does only provide the name of the article and no other informations like the author or the description. To get those informations we can use the helper function `{{ getPageMeta(settings,pageItem) }}` and fetch the meta-information for each article:

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

You can go even one step further and list articles of any folder on any page. You can also **list articles on the startpage** and create some kind of blog-design. Just use the `getPageList()` helper-function and inject the full navigation, the folder that should be listed and the base-url. Of course, you can get the desired folder from the theme-settings, so the author can choose the folder that he wants. Just check the following snippet from the [emergency theme](https://themes.typemill.net/emergency):

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

