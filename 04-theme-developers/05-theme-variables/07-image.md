# image

The Twig variable `{{ image }}` holds the image url and the image alt-text of the lead image of an article. Use it for image sharing in the meta-tags or as header image. The lead image is defined with the following logic:

* Use hero-image from meta tab.
* If not present, use first image from the content page.
* If not present, use logo. 
* If not present, return false.

## Example Usage

As a starting point to create a hero image for an article:

```
<header>
  <img url="{{ image.img_url }}" alt="image.img_alt">
  <h1>{{ title }}</h1>
</header>
```


You can also use the image for the meta-tags like below. Remember, that Typemill will already add this meta-information if you use the [renderMeta()](/theme-developers/theme-functions/rendermeta)-function in your theme.

````
<meta property="og:image" content="{{ image.img_url }}">
<meta name="twitter:image:alt" content="{{ image.img_alt }}">
````

