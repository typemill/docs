# base_url

The Twig-variable `{{ base_url }}` holds the base url of your website. Use it to create absolute urls to references.

## Example Usage

The base url is especially useful if you want to reference css or js-files like this:

````
<link rel="stylesheet" href="{{ base_url }}/themes/typemill/css/style.css" />
````

If you want to create buttons or links to the homepage, please use the [home url](/theme-developers/theme-variables/baseurl) instead.

