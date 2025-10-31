# Develop the Twig Template

The Twig-Template is ususally just one file with the name `index.twig`. If you want to create your own template, then you should copy and paste the existing demo-template and adopt it to your needs. We will explain some elements in the following.

## The Basic Layout

Typemill uses the library paged.js to generate a HTML-preview of the eBook. Paged.js is a polyfill to add paged media capacities to the (chrome-) browser. This means that coding an eBook layout is pretty the same as developing a Typemill theme. All you need to know is Twig, HTML, CSS and JavaScript. 

A basic eBook template looks like this. We added some comments so you understand what happens. 

````
<!-- Set the language attribute at the start -->
{% set language = ebookdata.language ? ebookdata.language : settings.langattr %}
<!DOCTYPE html>
<html lang="{{ language | default('en-gb') }}">
    <head>
        <meta charset="UTF-8">
        <title>eBook Preview</title>
                <!-- this is optional of course -->
        <meta name="description" content="{{ metatabs.meta.description }}" />
        <meta name="author" content="{{ metatabs.meta.author }}" />
        <meta name="generator" content="Typemill" />
                <!-- link to your CSS file or files -->
        <link rel="stylesheet" href="{{ base_url }}/plugins/ebooks/booklayouts/typemill/layout-digital.css" />
        <link rel="stylesheet" href="{{ base_url }}/plugins/ebooks/booklayouts/typemill/layout.css" />
                ...
                ...
    </head>
       <!-- we will print out the ebookdata as json object to use them with JavaScript later -->
    <body id="ebook" data-ebookdata="{{ ebookdata|json_encode|e('html_attr') }}">
                <!--  for now we only print out the pure ebook-content here -->
        <div class="bookcontent">
            {% for chapter in book %}
                                <!-- if it is a first level headline, then we wrap it in a section so it starts on a new page -->
                {% if chapter.level == 1 %}
                    <section class="chapter">
                        {{ chapter.content }}
                    </section>
                {% else %}
                    {{ chapter.content }}
                {% endif %}
            {% endfor %}
        </div>
                <!-- now we include the paged.js script that generates the preview -->
        <script src="{{ base_url }}/plugins/ebooks/public/paged.polyfill.js"></script>
    </body>
</html>
````

Some explanations: 

* At first set the language for the html document. We look for a language attribute in the ebook-data and we use the global language attribute for the website as fallback. 
* Then we create a simple head and include the css file. You can organize the css files as you want.
* We print out the ebook-data in json format in the body tag to use it with JavaScript. We will see an example later.
* Then we only print out the pure content of the chapters. Each h1-headline will be wrapped in a separate section tag because we want to place it on a new left page with CSS later.
* Finally we include the paged.js script before the closing body tag.

A very simple eBook is ready now. But it has no cover yet. So let us include it now. 

## Cover Background and Cover Image

For the two-color cover image and for the cover background image I add this css-snippet into the head of our template:

````
<style>
    :root {
        --primary-bg-color: {{ ebookdata.primarycolor ? ebookdata.primarycolor : 'cadetblue' }};
        --secondary-bg-color: {{ ebookdata.secondarycolor ? ebookdata.secondarycolor : 'cadetblue' }};
    }
    {% if ebookdata.coverimage %}
        .cover{
            background-image: url( {{ base_url }}/{{ ebookdata.coverimage }} );
        }
    {% endif %}
</style>
````

All we have to do now is to add some Twig-code just after the opening body-tag: 

````
<div class="cover">
    {% if not ebookdata.coverimageonly %}
        <div class="cover-top">
            <h1 class="cover-title">{{ ebookdata.title }}</h1>      
        </div>
        <div class="cover-bottom">
            <h2 class="cover-subtitle">{{ ebookdata.subtitle }}</h2>
            <p class="cover-author">{{ ebookdata.author }}</p>
            <p class="cover-edition">{{ ebookdata.edition }}</p>        
        </div>
    {% endif %}
</div>
````

This will add a cover page and use the background image as cover. The cover is devided into a top and a bottom area. Both will only be included if the user did not activate the "image only" option. The top area contains the title, the bottom area contains the subtitle, author and edition line. You can organize it differently, of course.

Let us add some more special pages like the imprint now.

## Imprint Page

We only need a simple Twig snippet to integrate the new imprint page right after the cover page: 

````
<div class="imprint">
    {% set booklayout = ebookdata.booklayouts[ebookdata.layout] %}
    <div class="imprint-top">
        <p>Book layout: © {{ booklayout.Name }} ({{ booklayout.Link }}) Licence: {{ booklayout.Licence }}<br/>
        Created with https://typemill.net</p>
    </div>
    <div class="imprint-bottom">
        {{ markdown(ebookdata.imprint) }}
    </div>
</div>
````

This adds a top section with some standard information and a bottom section with the imprint text from the user. Again, this is a matter of taste and you can organize it differently, of course.

## Table of Contents

We can add a fully automatted table of contents into the eBook. This is a bit more complex but you will master it with this steps and the first twig-part for the table of contents is super easy: 

````
{% if ebookdata.toc %}
    <div class="tableofcontents">
        <h1>{{ ebookdata.toctitle }}</h1>
        <div id="toc"></div>
    </div>
{% endif %}
````

This snippets just adds a div class with the title and an empty div with the id "toc", if the table of contents has been activated by the user. We will replace the div with the id "toc" with some JavaScript now. 

First of all we want to make the ebookdata that we added to the body-tag accessible for JavaScript, so we will parse it like this: 

````
<script>
    let ebookconfig = document.querySelector('#ebook');
    ebookdata = JSON.parse(ebookconfig.dataset.ebookdata);
</script>
````

Then we will extend paged.js and inject the table of contents like this:

````
<script>
    class handlers extends Paged.Handler {
        constructor(chunker, polisher, caller) {
            super(chunker, polisher, caller);
        }
        beforeParsed(content){
            if(ebookdata.toc){
                createToc({
                    content: content,
                    tocElement: '#toc',
                    titleElements: tocHeadlines
                });
            }
        }
    }
    Paged.registerHandlers(handlers);       
</script>
````

As you can see you have to add the id "#toc" as the `tocElement` and an array with all headline elements as the `titleElement`. 

Finally we will add the JavaScript that generates the Table of Contents and create the array of tocHeadlines that we added above: 

````
{% if ebookdata.toc %}
    <script src="{{ base_url }}/plugins/ebooks/public/paged-toc.js"></script>
    <script>     
        var tocHeadlines = [];
        var tocLevels = ebookdata.toclevel;
        for(var i = 1; i <= tocLevels; i++)
        {
            tocHeadlines.push('.bookcontent h'+i);
        }
    </script>
{% endif %}
````

If you provide the option to display a counter before each entry in the TOC, then you have to add some special css in the head to make it work. Simply copy and paste this code: 

````
        {% if ebookdata.toccounter %}
            <style>
                /* add chapter/headline counters before headline text */
                #list-toc-generated{ 
                    counter-reset: counterTocLevel1; 
                }
                #list-toc-generated .toc-element-level-1{ 
                    counter-increment: counterTocLevel1; 
                    counter-reset: counterTocLevel2; 
                }
                #list-toc-generated .toc-element-level-2{ 
                    counter-reset: counterTocLevel3; 
                }
                #list-toc-generated .toc-element-level-1::before{ 
                    content: counter(counterTocLevel1) ".";
                    padding-right: 5px;
                }
                #list-toc-generated .toc-element-level-2{ 
                    counter-increment: counterTocLevel2; 
                }
                #list-toc-generated .toc-element-level-2::before{ 
                    content: counter(counterTocLevel1) "." counter(counterTocLevel2);
                    padding-right: 5px;
                }
                #list-toc-generated .toc-element-level-3{ 
                    counter-increment: counterTocLevel3; 
                }
                #list-toc-generated .toc-element-level-3::before{ 
                    content: counter(counterTocLevel1) "." counter(counterTocLevel2) "." counter(counterTocLevel3);
                    padding-right: 5px;
                }
                #list-toc-generated .toc-element-level-4{ 
                    counter-increment: counterTocLevel4; 
                }
                #list-toc-generated .toc-element-level-4::before{ 
                    content: counter(counterTocLevel1) "." counter(counterTocLevel2) "." counter(counterTocLevel3) "." counter(counterTocLevel4);
                    padding-right: 5px;
                }
            </style>
        {% endif %}
````

### eBook Data in the Body Tag

We recommend to add the eBook-data as json-object in the body tag, so you can use it in the following scripts. Just copy and paste:

````
<body id="ebook" data-ebookdata="{{ ebookdata|json_encode|e('html_attr') }}">
````

### eBook Content

We won't explain all the Twig-code for the eBook content because it should be pretty clear. Simply check the demo layout and copy and paste what you need. As an example, this is the (pretty self-explanatory) code to print out each chapter of the ebook: 

````
        <div class="bookcontent">
            {% for chapter in book %}
                {% if chapter.level == 1 %}
                    <section class="chapter">
                        {{ chapter.content }}
                    </section>
                {% else %}
                    {{ chapter.content }}
                {% endif %}
            {% endfor %}
        </div>
````

It will wrap all first level chapters in a separate `section`-tag that ensures, that the main chapters start on a new page. You can do that differently, of course.

## Endnotes

You can transform footnotes into endnotes if you want. To do so simply add the following lines after the content loop:

````
{% if ebookdata.endnotes %}
    <div id="endnotes">
        <h1>{{ebookdata.endnotestitle}}</h1>
    </div>
{% endif %}
````

The JavaScript part is pretty simple too, just add the endnotes script and initialize it like this:

````
{% if ebookdata.endnotes %}
    <script src="{{ base_url }}/plugins/ebooks/public/endnotes.js"></script>
    <script>
        endnotes('#endnotes');
    </script>
{% endif %}
````

## Blurb

The blurb is just a separate page with some markdown content. Simply add this script:

````
{% if ebookdata.blurb %}
    <div class="blurb">
        {{ markdown(ebookdata.blurb) }}
    </div>
{% endif %}
````

