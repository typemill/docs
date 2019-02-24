# Public Forms

Public forms for contact or newsletter subscriptions are a common requirement for websites, but often a hassle for developers. Typemill makes your live much easier, because all you need is a plugin with some form-definitions in YAML and a business logic with php. All the complicated stuff like generating the frontend-forms, validating form-data and adding spam security or CSRF-protection is done by Typemill automatically.

## Form Definitions

Public forms are defined in a seprate block of the YAML of your plugin. Public forms can use the exact same [field definitions](/for-plugin-developers/documentation/field-overview) like admin forms. Look at this example:

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

You probably want to customize the labels and text-hints of your public forms, so that the admin can localize them for example. This can be done by a simple reference like this:

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

The `label` of the public `mail`-field has a reference to the `mail_label` field of the admin forms. This way, the admin can overwrite the label for the mail-field of the public form in the plugin-settings. You can add references like this for three fieldtypes:

* label
* help
* description

## Business Logic

Typemill has a pretty straigh-forward logic for public forms: If a user sends data with a public form, then Typemill will check the data and redirect the user back to the original page. If the data are valid, then Typemill will store the data before it redirects the user. If the data are not valid and contain errors, then Typemill will add error messages before it redirects the user.

In your plugin, you can handle both, the frontend form AND the incomming form-data, with two handy methods. Both methods expect the name of the plugin as a variable:

* `$this->getFormdata('pluginName')`: Checks and returns input data from a form. 
* `$this->generateForm('pluginName')`: Generates the frontend-form.

If the user has submitted a form and if the form-data are valid, then the method `$this->getFormdata` will return the data and you can process the data. You always have to check for existing form-data first. If no form-data exists, then you can generate the frontend-form and display it to the user. So your code will always look similar to this:

````
# check if form data have been stored
 
$formdata = $this->getFormdata('contactform');
 
if($formdata)
{
    # process the formdata here, e.g. store them or send a mail
}
else
{
    # get the public forms for the plugin
    $contactform = $this->generateForm('contactform');				
}
````

## Spam Security

Spam is really annoying, so Typemill tries to exclude spam-bots from public forms. If a spam-bot is detected, then the method `$this->formdata('myplugin')` returns the keyword `bot`, so you should always add an extra if-statement for this: 

````
$formdata = $this->getFormdata('contactform');
if($formdata)
{
    if($formdata == 'bot')
    {
        # add a message for the frontend here
    }
    else
   {
        # process the form-data
   }
}
...
````

Be default Typemill adds a simple honeypot test to each form. Typemill also adds an option to activate google recaptcha for each form, so the author can decide in the settings of your plugin if he wants to use recaptcha protection or not. You don't have to care about that.

## Security and Validation

Next to the spam protection Typemill adds a CSFR-check to each form. Additionally, Typemill will validate all incoming data against the original form definition so no additional data can be posted. Incoming data are validated with common rules, no html or other code-characters will pass the validation. All error messages are added automatically, so you do not have to care about it. It is strongly recommended that you use HTTPS connections for all public forms.

