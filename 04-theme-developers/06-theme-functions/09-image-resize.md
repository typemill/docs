# image().resize()

With the Twig-function `{{ assets.image(string imgurl).resize(int width, int height).src() }}` you can resize or crop an image on the fly. This function is useful if you want to render a certain image size in the frontend, for example a list of news-boxes with preview images on the start-page.

## Parameter

| Parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| imgurl | string | required | A string with a relative path to an existing image. |
| width | int | optional | An number for the width in px. |
| height | int | optional | An number for the height in px. |

## Resize or Crop

* To resize an image, add an int-value to the width OR to the height. 
* To crop an image, add int-values to the width AND to the height.

## Return 

This function return the relative path to the resized image. Resized images are stored in the folder `/media/custom`

## Example Usage

In the following example the image is resized to 800 px width:

````Twig
<img src="{{ assets.image(metatabs.meta.heroimage).resize(800).src() }}" />
````

Resize an image to 500 px height:

```Twig
<img src="{{ assets.image(metatabs.meta.heroimage).resize(false,500).src() }}" />
```

Crop an image to 800px width and 500px height:

```Twig
<img src="{{ assets.image(metatabs.meta.heroimage).resize(800,500).src() }}" />
```

You can also chain the resize-method and the [grayscale-method](/theme-developers/theme-functions/image-grayscale).

## Parameter

* Param `$imgurl`: The relative link to the image resource in the media-library as string. 
* Param `$width`: The width of the resized image in pixel (int or false).
* Param `$height`: The height of the resized image in pixel (int or false).
* Return: The relative url to the resized image or false.

## Explanation

This Twig-method uses method chaining. You have to provide all three methods:

* `image($imgurl)`: To get the source image. 
* `resize($width, $height)`: To resize or crop the image. 
* `src()`: To generate the url to the resized image.

Typemill will first check, if the image is already in the folder  `/media/custom/` and return the url to the image. If the image is not there yet, it will generate the resized version and store it in folder  `/media/custom/`.

