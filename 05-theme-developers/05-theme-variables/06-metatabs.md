# metatabs

The Twig-variable `{{ metatabs }}` holds all informations added to the metatabs of a page as an array. All meta-tab-information are stored in the yaml-version of a content file.

## Example Usage

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

Plugins and themes can extend the metatabs with more informations and even create new tabs in the yaml-configuration-file. If a plugin developer adds a new tab called `mytab` with a checkbox called `mycheckbox`, then a theme developer can use that information like this:

````
{{ metatabs.mytab.mycheckbox }}
````

