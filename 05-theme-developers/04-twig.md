# Templates with Twig

Twig is a flexible, fast, and secure templating engine for PHP. Twig is used by many modern software projects and content management systems like CraftCMS, Statamic, and even Drupal. If you have never used a template language before, then there are some good reasons to start with it today:

- The Twig syntax is **much shorter** than PHP, so your templates look cleaner and are easier to maintain.
- Twig produces **fewer errors**. An unknown variable produces an error in PHP, but it does not in Twig. Twig handles most of these cases, so you skip a lot of ugly logic in your theme.
- Twig is very **widespread**, so you can work with Twig in many other CMS.

The full Twig documentation for template designers is just one page long, so just head [over to Twig's website](https://twig.symfony.com/doc/3.x/templates.html) and read it. You can learn the most important essentials of Twig for TYPEMILL in the following list.

## Basic Twig Syntax

In a Twig template, you can use ordinary HTML markup. Statements and expressions are written in curly brackets.

Twig uses two curly brackets **to print out** a variable or expression: 

````
<p>{{ variable }}</p>
````



Twig uses one curly bracket with a percent sign **to execute** statements such as loops:

````
<ul>
    {% for element in breadcrumb %}
      <li> {{ element.output|e }} </li>
    {% endfor %}
</ul>
````



As you can see, the Twig syntax is cleaner and easier than pure PHP:

- You don't need the long `<?php echo something; ?>` introduction.
- You don't need the `$` to mark a variable.
- You don't need the `;` to finish a statement.
- You don't need the `->` or `['foo']` notation for objects and arrays, just use a dot-notation like `element.name` for everything.
- You don't need a lot of `()` like `foreach(a as b)`.
- You don't need a syntax like `<?php echo htmlspecialchars($element->output, ENT_QUOTES, 'UTF-8') ?>` for escaping special characters; just use a filter with a pipe notation like this `{{ element.output|e }}`.

## References

These are some useful examples and snippets that you can use for your templates. 

### Simple Variable

Set a simple variable and print it out:

````
{% set content = "my content" %}
{{ content }}
````



### Array

Set an array and print out a value:

````
{% set content = ['first' => 'one value', 'second' => 'another value'] %}

{{ content.first }} // prints out 'one value'
````



### Object

Set an object and print out a value:

````
{% set content = {'first' : 'first value', 'second' : 'another value'} %}

{{ content.first }} // prints out 'first value'
````



### Loop

Loop over an object or array and print out the values:

````
{% for value in content %}
   {{ value }}
{% endfor %}
````



Outputs:

- first value
- another value

### Filters

Set the first character of the words to uppercase:

````
<ul>
{% for value in content %}
   <li>{{ value|title }}</li>
{% endfor %}
</ul>
````



Output:

- First Value
- Another Value

You can manipulate variables with filters. Filters are used after a pipe notation. See a list of all filters in the [Twig documentation](https://twig.symfony.com/doc/3.x/filters/index.html).

### Functions

Print out content that was created in the last 30 days: 

```
{% if date(content.created_at) > date('-30days') %}
    {{ content.title }}
{% endif %}
```



'created_at' could be a timestamp of the content file. See a list of all functions in the [Twig documentation](https://twig.symfony.com/doc/3.x/functions/index.html).

### Include Template

To include a template, just write:

````
{{ include(sidebar.twig) }}
````



**Example usage**: Your layout-template **includes** other templates like header.twig, footer.twig or sidebar.twig.

### Extend Template

To extend a template, just write:

````
{% extends "partials/layout.twig" %}
````



**Example usage:** Your content template (e.g. index.twig) **extends** your layout template. This means, index.twig is rendered within the layout.twig, and the layout.twig includes a header.twig, a footer.twig and a sidebar.twig.

### Example: Include and Extend

If you extend a template with another template (e.g. if you extend `layout.twig` with `index.twig`), then you have to define some content areas in the "parent" template that get overwritten with the content of the "child" template. You can use the "block" statement to define such areas. 

Your layout.twig looks like this:

````
<!DOCTYPE html>
<html>
  <head>
    ...
  </head>
  <body>
    <article>{% block content %}{% endblock %}</article>
    <nav>
      {{ inlude 'navigation.twig' }}
    </nav>
    <footer>
      {{ include 'footer.twig' }}
    </footer>
  </body>
</html>
````



Your index.twig looks like this:

````
{% extends "layout.twig" %}

{% block content %}

  <ul>
    {% for value in content%}
      <li>{{ value }}</li>
    {% endfor }
  </ul>

{% endblock %}
````



Now, the `index.twig` template extends the `layout.twig` template, and the `block content` in your layout template gets replaced by the `block content` defined in your index template. At the same time, the layout template includes the navigation and the footer.

### Macros

Macros in Twig are like functions in PHP: You can use them for repeating tasks. A typical example is a navigation, where you loop over a comlex and multi-dimensional array recursively. But you can also use macros to render forms and input fields.

Macros are usually the most complex parts of your theme, and they are mostly used for generating navigation. You can use the macro for the navigation used in the Typemill theme as a starting point. In most cases, you won't even touch it, but if you read the code in detail, it will probably pretty easy for you to costumize it for your needs.

A typical macro code for a navigation looks like this: 

    {% macro loop_over(navigation) %}
      {% import _self as macros %}
      {% for element in navigation %} 
        <li>
          {% if element.elementType == 'folder' %}
            <a href="{{ element.url }}">{{ element.name|title }}</a>        
            <ul>
              {{ macros.loop_over(element.folderContent) }}
            </ul>
          {% else %}
            <a href="{{ element.url }}">{{ element.name|title }}</a>
          {% endif %}
        </li>
       {% endfor %}
    {% endmacro %}
    {% import _self as macros %}
    <ul class="main-menu">
      {{ macros.loop_over(navigation) }}
    </ul>

These are only some small examples of how you can use Twig to create templates and themes for TYPEMILL. In fact, you can do a lot more complex stuff with Twig. Just read the [official documentation](https://twig.symfony.com/doc/3.x).

