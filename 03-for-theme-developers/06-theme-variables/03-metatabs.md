# MetaTabs

Authors can add a lot of additional meta-information to each page with metatabs. Metatabs are visible in the editor at the top of each page and they can contain unlimited forms and fields. This system is super flexible and similar to the well known concept of frontmatter in markdown. But different to frontmatter, Typemill stores all meta-informations for a page in a separate YAML file.

You have access to all those meta-informations in your theme. For example:

```
  {{ metatabs.meta.title }} // prints out the metatitle.
  {{ metatabs.meta.description }} // prints out the metadescription.
  {{ metatabs.meta.author }} // prints out the article author.
  {{ metatabs.meta.manualdate }} // prints out the manual date of the article if it exists.
  {{ metatabs.meta.modified }} // prints out the last modified date for the article.

```

The meta-informations are typically used for the meta-head of your theme:

````
<meta name="description" content="{{ metatabs.meta.description }}" />
````

You can also manipulate the description with Twig filters:

````
{{ metatabs.meta.description|slice(0,100) }}
````

This will output the first 100 characters of the description.

## Additional information and more tabs

Plugin-developers can extend the metatabs with more informations and even create new tabs. If a plugin developer adds a new tab called `mytab` with a checkbox called `mycheckbox`, then a theme developer can use that information like this:

````
{{ metatabs.mytab.mycheckbox }}
````
