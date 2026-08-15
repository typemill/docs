# navigation

The Twig-variable `{{ navigation }}` holds the structure of the whole content folder as an array of objects. It can be used to create [global navigations](/theme-developers/examples/create-navigations) on your page.

The `{{ navigation }}` variable is a multi dimensional array of item objects. With that array you have access to nearly all information that an [item object](/theme-developers/theme-variables/item) provides. Only the following information for the paging is not part of the item objects within the navigation variable:

- thisChapter
- nextItem
- prevItem

## Example of the {{ navigation }} variable 

This is an example of the `{{ navigation }}`  variable containing just one folder and a file:

    Array(
       [0] => stdClass Object(
           [originalName] => 0_about-typemill
           [elementType] => folder
           [index] => 1
           [order] => 0
           [name] => about typemill
           [slug] => about-typemill
           [path] => \0_about-typemill
           [urlRel] => /about-typemill
           [urlAbs] => http://localhost/about-typemill
           [key] => 0
           [keyPath] => 0
           [keyPathArray] => Array
           (
               [0] => 0
           )
           [chapter] => 1
           [folderContent] => Array
           (
               [0] => stdClass Object(
                    [originalName] => 02-what-is-mardown.md
                    [elementType] => file
                    [fileType] => md
                    [order] => 02
                    [name] => what is mardown
                    [slug] => what-is-mardown
                    [path] => \0_about-robodoc\02-what-is-mardown.md
                    [key] => 0
                    [keyPath] => 0.0
                    [keyPathArray] => Array
                    (
                        [0] => 0
                        [1] => 0
                    )
                    [chapter] => 1.1
                    [urlRel] => /about-robodoc/what-is-mardown
                    [urlAbs] => http://localhost/about-robodoc/what-is-mardown
                )
            )
        )
    )

