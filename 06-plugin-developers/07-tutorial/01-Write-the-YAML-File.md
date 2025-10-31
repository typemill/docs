# Write the YAML-Configuration-File

Every plugin should have a basic configuration file with some meta information: like the version, the license, and the author.

The configuration file is located in the root folder of your plugin, and must have the same name as the plugin folder. For the cookie-consent-plugin, it looks like this:

````
/cookieconsent
  - cookieconsent.yaml
  - cookieconsent.php
````

The configuration file is written in simple YAML syntax. The basic YAML file for the Cookie Consent plugin looks like this:

````
name: Cookie Consent
version: 1.0.0
description: Enables a cookie consent for websites
author: Sebastian Schürmanns
homepage: https://cookieconsent.insites.com/
licence: MIT
````

## YAML Transforms to Arrays

If you have never heared about YAML: YAML is a simple configuration language; with a library you can easily transform a YAML file into an array. That is why YAML is so widespread. In TYPEMILL, the basic properties defined in YAML are transformed into a simple one-dimensional array like this:

````
array (
  'name' => 'Cookie Consent,
  'version => '1.0.0',
  'description' => Enables a cookie consent for websites',
  'author' => 'Sebastian Schürmanns',
  'homepage' => 'https://cookieconsent.insites.com/',
  'licence' => 'MIT'
)
````

Of course, you can also create a multi-dimensional array. This works with simple indentation; use two spaces to indicate the next level. A YAML definition like this:

````
options:
  first: yellow
  second: green
````

Would transform to this array:

````
array(
  'options' => array(
  	'first' => 'yellow',
  	'second' => 'green
  )
)
````

When you write YAML, be careful to only use spaces for indentation. If you use the tab key, then the transformation will break.

## More Configurations

For now, we have only some basic settings in your configuration file, but there are more possibilities, and we will learn about them later. For now, just remember that a configuration file can have three parts, depending on the plugin's needs:

* Basic information about the plugin.
* Default values for the plugin.
* Definitions for input fields, so that the user can change the default values in the setup.

## Background

In theory, a very simple plugin can work without a configuration file. But it is highly recommended to add a configuration file to any plugin. Here are the reasons:

* TYPMILL uses the version number in the configuration file to check if there is a newer version of the plugin. So this is pretty important.
* The settings in the configuration file are displayed to the user in the setup of TYPEMILL. So this is pretty important, too.

## Next: Basic PHP-File

The configuration file was a pretty easy start. In the next chapter, we will get our hands dirty and write the first bit of PHP code.