# Image Manipulation

The author-interface of Typemill provides image uploads for meta-tabs, themes and plugins. There is only one image upload field by default: The hero-image that authors can add to each page in the meta-tab. 

You can integrate the hero-image, or any other image that has been uploaded to meta-tabs, themes or plugins, with a simple twig-reference like this: 

````
<img src="{{ metatabs.meta.heroimage }}" />
````

This will add the image with the default image width of 820px. 

Starting with version 1.3.7 you can also manipulate images on the fly with the asset asset-tag like this: 

````
<img src="{{ assets.image(metatabs.meta.heroimage).resize(800,500).src() }}" />
````

Typemill will generate the image size with the `resize(width,height)`-method on the fly and store the new image in the folder `/media/custom/`. If you add a value for `width` and `height`, then Typemill will resize and crop the image accordingly. If you only want to resize the image, then add a `false` to the other value like this: `resize(400,false)`. 

You can also transform a colored image into a grayscale image with the `grayscale()`-method like this: 

 ````
<img src="{{ assets.image(metatabs.meta.heroimage).grayscale().src() }}" />
````

And finally you can resize and grayscale an image in one line like this: 

````
<img src="{{ assets.image(metatabs.meta.heroimage).resize(800,500).grayscale().src() }}" />
````

