# Shortcodes

Shortcodes are small snippets that an author can add to a markdown-page. Shortcodes are a super easy and smart way to add any kind of content or feature to a page. The syntax of a shortcode looks like this:

```
[:shortcodename param1="myvalue" param2="anothervalue" :]
```

When the markdown is transformed into html, then Typemill will look for such shortcodes and fire the event `onShortcodeFound`. You can listen to that event, analyse the params of the shortcode and then return any kind of content or feature as HTML. As of version 1.5.2 you can also register our shortcode so the author can use the visual editor to add your shortcode more easily.

## Example 1: A replacement shortcode

Let us quickly create the shortcode `[:replace:]` that gets replaced with some html. The plugin looks like this:

```php
<?php

namespace Plugins\replace;

use \Typemill\Plugin;

class replace extends Plugin
{

  protected $settings;

  # subscribe to the shortcode event
  public static function getSubscribedEvents()
  {
    return array(
      'onShortcodeFound' => 'onShortcodeFound'
    );
  }

  # listen, when typemill found a shortcode and fires the event
  public function onShortcodeFound($shortcode)
  {
    # read the data of the shortcode
    $shortcodeArray = $shortcode->getData();

    # check if it is the shortcode name that we where looking for
    if(is_array($shortcodeArray) && $shortcodeArray['name'] == 'replace')
    {
      # we found our shortcode, so stop firing the event to other plugins
      $shortcode->stopPropagation();

      # and return a html-snippet that replaces the shortcode on the page.
      $shortcode->setData('<span class="tm-green">A replacement from the plugin</span>');
    }
  }
}
```

Just read the code with the comments and you will easily understand:

* We subscribe to the event onShortcodeFound.
* Typemill finds a shortcode and fires the event.
* We listen to that event and check the name of the shortcode.
* If the shortcode has the name we where looking for, then we simply return some html.

Now the author can insert the shortcode `[:replace:]` to the markdown of a page and he will see the replacement from your plugin.

## Example 2: Replace with params

So far so good, but we can make it much more useful. Let us now replace the shortcode and use a text with a color. We need the two params `text` and `color` for that so a shortcode might look like this: 

```
[:replace text="I am a red text" color="red":]
```

In our plugin we can now return the text with the color like this:

```php
<?php

namespace Plugins\replace;

use \Typemill\Plugin;

class replace extends Plugin
{

  protected $settings;

  # listen to the shortcode event
  public static function getSubscribedEvents()
  {
    return array(
      'onShortcodeFound' => 'onShortcodeFound'
    );
  }

  # if typemill found a shortcode and fires the event
  public function onShortcodeFound($shortcode)
  {
    # read the data of the shortcode
    $shortcodeArray = $shortcode->getData();

    # check if it is the shortcode name that we where looking for
    if(is_array($shortcodeArray) && $shortcodeArray['name'] == 'replace')
    {
      # we found our shortcode, so stop firing the event to other plugins
      $shortcode->stopPropagation();

      # Of course you should validate the user input here, but let us skip it to keep it easy ...
      $text = isset($shortcodeArray['params']['text']) ? $shortcodeArray['params']['text'] : 'Fallback text';
      $color = isset($shortcodeArray['params']['color']) ? $shortcodeArray['params']['color'] : 'green';
      $html = '<span style="color:' . $color . ';">' . $text . '</span>';

      # and return a html-snippet that replaces the shortcode on the page.
      $shortcode->setData($html);
    }
  }
}
```

That is all you need to know to create a new shortcode and extend the content area with anything you can imagine. In this simple case we only change the color of a text. If you want to see a more complex shortcode, then you can check the plugin [ebookproducts](https://plugins.typemill.net/ebookproducts). With that plugin, the user can create multiple product-boxes for eBooks and insert a product box with a shortcode into any page .

## Example 3: Register your shortcode

As of Typemill version 1.5.2 you can also register your shortcode for the visual editor. If you do so, the author can easily select your shortcode from a shortcode-list and even search existing values for each parameter of your shortcode. 

![screenshot of the shortcode interface in the visual editor](media/live/visual-shortcodes.png){.center loading="lazy" width="891" height="226"}

This is especially helpful if your shortcode handels a lot of data. To see how it works and how it is coded, please check the plugin for [ebookproducts](https://plugins.typemill.net/ebookproducts).

To register your shortcode, simply listen to a shortcode with the name "registershortcode" like this: 

```
if(is_array($shortcodeArray) && $shortcodeArray['name'] == 'registershortcode')
{
   # here you can register your shortcode
}

```

You can simply return the structure of your shortcode and the content of your shortcode here. The structure means: 

* The name of your shortcode.
* The parameter of your shortcode.

The shortcode for the eBook-products has the name "ebookproduct" and it has only one parameter called "id". So simply add it to the shortcode-array with the name "data" like this:

```
if(is_array($shortcodeArray) && $shortcodeArray['name'] == 'registershortcode')
{
    $shortcodeArray['data']['ebookproduct'] = [ 'id' ];

    $shortcode->setData($shortcodeArray);
}
```

Now the visual editor will add your name "ebookproducts" to the list of shortcodes that the user can select, and if the user selected your shortcode, the editor will show an input field for the id. 

In the next step you can add existing data for your shortcode-id, so the user can search for that data in the id-field. To do so, simply add the data to the id with a new array that has the key-name "content" and another array with the content-strings that the user can search for: 

```
if(is_array($shortcodeArray) && $shortcodeArray['name'] == 'registershortcode')
{
    $settings       = $this->getSettings();
    $folderName     = 'data' . DIRECTORY_SEPARATOR . 'ebookproducts';
    $folder         = $settings['rootPath'] . $folderName;
    $writeYaml      = new WriteYaml();
    $ebookproducts  = $writeYaml->getYaml($folderName, 'ebookproducts.yaml');
    $content        = [];

    if($ebookproducts)
    {
        foreach($ebookproducts as $key => $value)
        {
            $content[] = $key;
        }
    }

    $shortcodeArray['data']['ebookproduct'] = [ 'id' => ['content' => $content] ];

    $shortcode->setData($shortcodeArray);
}
```

It is important to keep the structure of the data consistent, otherwise it will break. So please be careful and test your shortcode registration carefully before you deploy it.

