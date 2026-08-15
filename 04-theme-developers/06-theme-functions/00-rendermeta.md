# renderMeta()

The twig-function `{{ assets.renderMeta() }}` adds standard meta-tags to the theme. Use the plugin methods [getMeta()](/plugin-developers/method-overview#h-this-addmetakey-meta) and [addMeta()](/plugin-developers/method-overview#h-this-addmetakey-meta) to add or remove meta-tags.

## Example Usage

````Twig
<head> 
  <title>{% block title %}{% endblock %}</title>

  <meta name="description" content="{{ metatabs.meta.description }}" />
  <meta name="author" content="{{ metatabs.meta.author }}" />
  <meta name="generator" content="Typemill" /> 
  <meta rel="canonical" href="{{ item.urlAbs }}" /> 

  {{ assets.renderMeta() }}
</head>
````



## Standard Meta Tags

By default Typemill will add the following meta-tags:

* `og:site_name`
* `og:title`
* `og:description`
* `og:type`
* `og:url`

Typemill will add the following meta-tags, if the admin has added these information in the admin panel:

* `icon` (Favicons in different sizes)
* `noindex`
* `og:image`
* `twitter:image:alt`
* `twitter:card`

It will **not add** meta-data like title, description, or canonical tags, so you have to add these meta-tags manually.

