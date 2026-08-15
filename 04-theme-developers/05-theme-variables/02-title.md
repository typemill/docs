# title

The Twig-variable `{{ title }}` returns the first `<h1>` headline used in the content file as unformatted text-string (no html or markdown syntax). If there is no headline, it uses the file name.

## Example Usage

````
{{ title }}
````





You can use the title for the HTML `<title>` like this: 

````
<title>{{ title }}</title>
````





And you can of course manipulate the title with a Twig filter like this: 

````
{{ title|title }}
````





This will display the first character of each word in uppercase.

Also consider to use the title tag from the [{{ metatabs }}](/theme-developers/theme-variables/metatabs).

