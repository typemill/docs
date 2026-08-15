# validateParams

The method `$this->validateParams($params)` validates parameters, for example the input-params of a form send with a post-request. It uses standard validation based on field-types. Additionally you can add validation rules with regex in the form-definitions of your yaml-configuration-file.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------|
| $params  | array              | Yes      | The parameters to validate, usually retrieved from the slim request. |

## Return

* `mixed` with the validated parameters if successful
* `false` if validation failed.

## Example Usage

```php
<?php

namespace Plugins\Myplugin;

use \Typemill\Plugin;

class Myplugin extends Plugin
{
    ...
    public function getforminput(Request $request, Response $response, $args)
    {
        $forminput = $request->getParsedBody();
        $validvalues = $this->validateParams($forminput);
        if(!$validvalues)
        {
            # errors are added to session automatically
            # here you could send user to an error page
           # or get the referrer, redirect to the referrer and show errors in form
        }

        # here you can do something with the validvalues from the form
    }
}
```

See chapter about [public forms](/forms/public-forms) for more details. Check the plugin [contactform](https://plugins.typemill.net/contactform) as a first starting point how to work with public forms.

