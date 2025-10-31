# onSystemnaviLoaded

This event is triggered when the system navi gets loaded. Use it to add a new navigation item.

## Availability

This event is available in the system view of the admin interface.

## Data

Plugins **must** return an array with the system navigation for this event; otherwise, the system navigation will not be available.

| Getter/Setter | Type | Required | Description | 
|:---|:---|:---|:---|
| getData | array | - | Two-dimensional array of system navigation items. | 
| setData | array | required | Initial or manipulated array of system navigation items. |

## Example Data

```
Array
(
    [system] => Array
        (
            [title] => System
            [routename] => settings.show
            [icon] => icon-wrench
            [aclresource] => system
            [aclprivilege] => read
        )

    [license] => Array
        (
            [title] => License
            [routename] => license.show
            [icon] => icon-wrench
            [aclresource] => user
            [aclprivilege] => read
        )

    [themes] => Array
        (
            [title] => Themes
            [routename] => themes.show
            [icon] => icon-paint-brush
            [aclresource] => system
            [aclprivilege] => read
        )

    [plugins] => Array
        (
            [title] => Plugins
            [routename] => plugins.show
            [icon] => icon-plug
            [aclresource] => system
            [aclprivilege] => read
        )

    [account] => Array
        (
            [title] => Account
            [routename] => user.account
            [icon] => icon-user
            [aclresource] => account
            [aclprivilege] => read
        )

    [users] => Array
        (
            [title] => Users
            [routename] => users.show
            [icon] => icon-group
            [aclresource] => user
            [aclprivilege] => read
        )

)

```

## Example Usage

```php
<?php

namespace plugins\myplugin;

use \typemill\plugin;

class myplugin extends plugin
{
    public static function getSubscribedEvents()
    {
        return [
            'onSystemnaviLoaded' => 'onSystemnaviLoaded'
        ];
    }

    # Add ebook-item to system navigation
    public function onSystemnaviLoaded($navidata)
    {
        $this->addSvgSymbol('<symbol id="icon-download" viewBox="0 0 24 24"><path d="M20 15v4c0 0.276-0.111 0.525-0.293 0.707s-0.431 0.293-0.707 0.293h-14c-0.276 0-0.525-0.111-0.707-0.293s-0.293-0.431-0.293-0.707v-4c0-0.552-0.448-1-1-1s-1 0.448-1 1v4c0 0.828 0.337 1.58 0.879 2.121s1.293 0.879 2.121 0.879h14c0.828 0 1.58-0.337 2.121-0.879s0.879-1.293 0.879-2.121v-4c0-0.552-0.448-1-1-1s-1 0.448-1 1zM13 12.586v-9.586c0-0.552-0.448-1-1-1s-1 0.448-1 1v9.586l-3.293-3.293c-0.391-0.391-1.024-0.391-1.414 0s-0.391 1.024 0 1.414l5 5c0.092 0.092 0.202 0.166 0.324 0.217s0.253 0.076 0.383 0.076c0.256 0 0.512-0.098 0.707-0.293l5-5c0.391-0.391 0.391-1.024 0-1.414s-1.024-0.391-1.414 0z"></path></symbol>');

        $navi = $navidata->getData();

        $navi['Ebookproducts']  = [
            'title'         => 'Ebookproducts',
            'routename'     => 'ebookproducts.show', 
            'icon'          => 'icon-download', 
            'aclresource'   => 'system', 
            'aclprivilege'  => 'view'
        ];

        # if the use visits the system page of the plugin
        if(trim($this->route,"/") == 'tm/ebookproducts')
        {
            # set the navigation item active
            $navi['Ebookproducts']['active'] = true;

            # add the system application
            $this->addJS('/ebookproducts/js/vue-ebookproducts.js');         
        }

        $navidata->setData($navi);
    }
}
```

## See also

* [Example](/plugin-developers/examples/system-navigation) how to change the system navigation for plugin developers.

