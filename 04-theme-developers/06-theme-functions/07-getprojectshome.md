# getProjectsHome

The Twig-function `getProjectsHome(settings)` returns all meta-data of the startpages from projects. The function is useful if you want to list projects on the homepage or on any other page.

## Parameter

| Parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| settings | array | required | The [settings](/theme-developers/theme-variables/settings) variable.  |

## Return 

This function returns an array of projects with metadata of the projects startpage.

## Example Usage

```
  {% set projectshome = getProjectsHome(settings) %}

  <div>
    {% for projectid,projectmeta in projecthome %}

      <div class="projectbox">
         <h2>{{projectmeta.meta.title}}</h2>
         <p>{{projectmeta.meta.description}}</p>
         <a href="{{ base_url }}/{{ projectid }}">Read more</a>
      </div>

    {% endfor %}
  </div>
```

