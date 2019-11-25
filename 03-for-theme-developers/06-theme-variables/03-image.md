# Image

If present, the image tag will return the url and the `alt=""` information of the first image used in the article: 

````
{{ image.img_url }}
{{ image.img_alt }}
````

This can be pretty handy if you want to use a header image, or if you want to add meta tags for social media networks. The Typemill standard theme uses meta tags for Twitter and Facebook, so that the image gets displayed in the social media posts. It can look like this: 

````
<meta property="og:image" content="{{ image.img_url }}">
<meta name="twitter:image:alt" content="{{ image.img_alt }}">
````

