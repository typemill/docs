# Public Forms

Public forms, like contact or newsletter subscription forms, are a common requirement for websites, but often a hassle for developers. Typemill makes your life much easier, because all you need is a plugin with some form definitions in YAML, and business logic in PHP. All the complicated stuff like generating the front end forms, validating, form-data, and adding spam security or CSRF-protection is done by Typemill automatically.

## Form Definitions

Public forms are defined in a separate block of the YAML-configuration-file of your plugin. Public forms can use the exact same [field definitions](/for-plugin-developers/documentation/field-overview) as theme- and plugin-forms. Look at this example:

````
forms:
  fields:
    myfield:
      type: text
      label: 'I am a textfield'
      required: true
public:
  fields:
    mypublicfield:
      type: text
      label: 'I am a textfield'
      required: true
````


If you want to provide an option for the admin to change the button-label for the form, then add the input field `button_label` to your plugin like this:

```
forms:
  fields:
    button_label:
      type: text
      label: Label for Register Button
      placeholder: Register now
      required: true
```


You probably want to customize the labels and text hints of your public forms, so that the admin can localize them for example. This can be done by a simple reference like this:

````
forms:
  fields:
    mail_label:
      type: text
      label: 'Label of Mail-Input-Field'
      placeholder: 'Localize label here'
      required: true
public:
  fields:
    mail:
      type: text
      label: mail_label
      required: true
````


The `label` of the public `mail` field has a reference to the `mail_label` field of the admin forms. This way, the admin can overwrite the label for the mail-field of the public form in the plugin-settings. You can add references like this for three fieldtypes:

* label
* help
* description

## Public Form Definitions

As of version 1.5.1 you can also provide a textarea with the name `publicformdefinitions`. In this textarea, the admin can define his own forms with the YAML syntax. 

```
forms:
  fields:
    publicformdefinitions: 
      type: textarea
      label: configure your public form with yaml
```


This is pretty useful, for example if you create a contactform plugin and if you want to provide the option to define an individual contact form with all kind of fields. But this also requires some more skills from the admin: He should understand YAML and he should understand at least a bit of HTML so he knows which fields exist  and which attributes they can have.

The yaml-definition from the textarea `publicformdefinitions` will overwrite all public form definitions that you made in the yaml file of your plugin. This can be a bit risky, for example if your plugin logic expects a field with the name "username" or "password". If the admin deletes those fields, the plugin will not work anymore.

It is a good idea to provide some standard-definitions for the publicformdefinitions in the settings of your plugin. This way, the user can see the standard definitions and even delete his own definitions later to switch back to the standard-definitions.

```
settings: 
  publicformdefinitions: "username:\r\n  type: text\r\n  label: username\r\n  placeholder: Username\r\n  required: true\r\n\r\nemail:\r\n  type: text\r\n  label: E-Mail\r\n  placeholder: Email\r\n  required: true\r\n\r\npassword:\r\n  type: password\r\n  label: Password\r\n  required: true\r\n\r\ngumroad:\r\n  type: password\r\n  label: Gumroad Licence Key\r\n  required: true"
```


## Business Logic

Typemill provides some handy methods to make form-handling as easy as possible. You will only need two new routes:

* A route where you want to show your form.
* A route where you want to process the form-data.

Let us check the [register-plugin](https://plugins.typemill.net/register) and use the routes from there: 

```
    public static function addNewRoutes()
    {
        return [
            ['httpMethod' => 'get', 'route' => '/tm/register', 'class' => 'Plugins\Register\Register:showRegistrationForm', 'name' => 'register.show'],
            ['httpMethod' => 'post', 'route' => '/tm/register', 'class' => 'Plugins\Register\Register:createUser', 'name' => 'register.create'],
        ];
    }
```


In the next step, let us create the method `showRegistrationForm` to display the form in the frontend. We can easily generate a form with the method `$this->generateForm()` like this:

```
public function showRegistrationForm($request, $response, $args)
{
            $twig   = $this->getTwig();  // ge the twig-object
        $loader = $twig->getLoader();  // get the twig-template-loader  
        $loader->addPath(__DIR__ . '/templates');
        $settings = $this->getSettings();
        # get the public forms for the plugin
        $registerform = $this->generateForm('register', 'register.create');
        return $twig->render($response, '/register.twig', ['settings' => $settings, 'registerform' => $registerform ]);
}
```


All you have to add here is the name of the plugin (`register`) and the name of the route where you want to send the data. In this case it is our second post-route with the name `register.create`.

Now let us create that method to handle the form-data. 

````
# create a new user
public function createUser($request, $response, $args)
{       
    $params = $this->validateParams($request->getParams());
    if(!$params)
    {
        return $response->withRedirect($this->container['router']->pathFor('register.show'));
    }
    return $response->withRedirect($this->container['router']->pathFor('welcome'));
}
````


This is super simplified but it is more or less all you have to do. The method `$this->validateParams($request->getParams())` will validate all incoming params from the user against the original form-definitions and it will return only the relevant params and strip out security-related params like csrf and more. You can, of course, do more validations if you want. And you probably want to do something with the params like store them somewhere. All this depends on the logic of your plugin.

## Security

Public forms have a lot of security implications. To make your forms as secure as possible, Typemill provides some build-in security features. At the same time a plugin developer and a website administrator should do everything on their own to protect their page.

### Use HTTPS

It is generally recommended to use HTTPS connections for websites. If you use public forms, then it is also often required by law (e.g. contact forms and dsgvo).

### Input Validation

The developer is responsible to validate all incoming params from the form. To make this easy as possible, Typemill provides the method `$this->validateParams($request->getParams())`. The method will validate all incoming data against the original form definition, so no additional data can be posted. Incoming data is validated with common rules; no HTML or other code characters will pass the validation. All error messages are added automatically, so you do not have to worry about it.

You can also add a regular expression to each field. The input data will be validated against this regular expression before the data is stored.

### CSRF Protection

Typemill automatically adds a csrf-protection to all forms, public and non-public. A csrf-protection makes sure that 

### Honeypot

Typemill will automatically add a honeypot field to all public forms. A honeypot field is a input field (text) that is visually hidden in frontend. If a bot fills it out (because it does not notice that it is hidden), then the bot will automatically redirected to the startpage after submitting the form.

### Captcha

Typemill will automatically add a traditional captcha field with a text-image to all public forms. A user can only submit a form if he fills out the captcha correctly. If the captcha is incorrect, then the user will be redirected to the form again and the data are not stored. This is a standard measure to protect forms from spam-bots and other techniques.

In your plugin you can integrate the option to:

* disable the captcha completely
* to show the captcha only after a wrong initial input.

To do so, please add the following radio-buttons in your plugin settings: 

```
forms:
  fields:
    captchaoptions:
      type: radio
      label: Configure the captcha
      options:
        standard: Show directly (standard)
        aftererror: Show after first wrong input
        disabled: Disable
```


You can change the field label and the description of each option. But you have to keep the fieldname `captchaoptions` and the option-names `standard`, `aftererror` and `disabled`. Otherwise it will be ignored and the captcha will always show directly.

### Google Re-Captcha

The traditional captcha field that ships with typemill (gregwar php captcha) does not rely on any external service and is a perfect solution without any privacy concerns. However, it is a bit difficult to read and solve and thus might lower the conversion of your forms. Also the protection might be lower because an image captcha can be solved by KI machines today.

As an alternative you can also provide a google recaptcha where the user only has to check a checkbox in most cases. Be aware that it might have some privacy implications when you add it to a website. 

To add an option for google re-captcha in your plugin settings, please add the following fields: 

```
forms:
  fields:
    recaptcha:
      type: checkbox
      label: Google Recaptcha
      checkboxlabel: Activate Recaptcha
    recaptcha_webkey:
      type: text
      label: Recaptcha Website Key
      help: Add the recaptcha website key here. You can get the key from the recaptcha website.
      description: The website key is mandatory if you activate the recaptcha field
    recaptcha_secretkey:
      type: text
      label: Recaptcha Secret Key
      help: Add the recaptcha secret key here. You can get the key from the recaptcha website.
      description: The secret key is mandatory if you activate the recaptcha field
```


If you add google recaptcha to your plugin, you also have to add the standard php-captcha option so the admin can disable it in the plugin settings. Otherwise you get two captchas.

### Track Suspicious Actions

In the developer settings of the admin area you can activate a feature that logs all (or most) actions that might result from attacks or spam bots. The log contains the IP address, the time and the action (like wrong captcha, wrong password, and more). You can activate and deactivate the log and you can delete the log with the button "recreate cache". 

### IP-blocking

This feature is not implemented yet, but it is planned for a future release of typemill.

### Individual Security Features for Plugins

If you develop a plugin, then think about additional security features. For example, if you create a registration feature, then you can block all registration attempts with so called burner mails. This is implemented with the existing register plugin. Another security feature from that plugin is the double opt in method.

