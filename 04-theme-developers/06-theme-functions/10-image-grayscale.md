# image().grayscale()

With the Twig-function `{{ assets.image(string imgurl).grayscale().src() }}` you can grayscale an image on the fly. This is useful if you want to render a grayscaled version of images in the frontend, for example a list of news-boxes with preview images on the start-page.

## Parameter

| Parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| imgurl | string | required | A string with a relative path to an existing image. |

## Return 

This function return the relative path to the grayscaled image. Grayscaled images are stored in the folder `/media/custom`

## Example Usage

Grayscale an image:

````Twig
<img src="{{ assets.image(metatabs.meta.heroimage).graysacle().src() }}" />
````

Resize and grayscale an image:

```Twig
<img src="{{ assets.image(metatabs.meta.heroimage).resize(false,500).grayscale().src() }}" />
```

