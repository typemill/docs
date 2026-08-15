# generateForm

The method `$this->generateForm($routename)` generates a form from the form-definitions in the plugin yaml configuration.

## Parameter

| Parameter  | Type                | Required | Description                                                                                 |
|------------|---------------------|----------|---------------------------------------------------------------------------------------------|
| $routename  | string              | Yes      | The name of the route.                                                                     |

## Return

* `string` with the generated form if successful.
* `false` if form generation failed.

## Example Usage

```php
<?php

namespace Plugins\Myplugin;

use \Typemill\Plugin;

class Myplugin extends Plugin
{
    ...

    public function myFunction()
    {
        # Data will be send as post-request to the route with the name 'contact.send'
        $form = $this->generateForm('contact.send');
    }
}
```

A public form definition in the plugin yaml file might look like this: 

```yaml

public:
  fields:

    name:
      type: text
      label: name_label
      required: true

    email:
      type: email
      label: email_label
      required: true

    subject:
      type: text
      label: subject_label
      required: true

    message:
      type: textarea
      label: message_label
      required: true
```

See chapter about [public forms](/forms/public-forms) for more details. Check the plugin [contactform](https://plugins.typemill.net/contactform) as a first starting point how to work with public forms.

