# HTML Plugin

Typemill restricts HTML writing in Markdown for security reasons. However, by using a plugin, you can incorporate HTML tags like div, span, address, mark, sub, sup, and kbd through shortcodes. Note that this plugin requires a MAKER License.

[:video path="media/files/typemill-html-plugin.webm" width="100%" preload="none" poster="media/live/typemill-html-plugin-poster.webp" :]

## Usage

You can utilize HTML tags in both the raw Markdown and visual editors. In the visual editor, you can add HTML tags to a paragraph element, allowing you to write HTML seamlessly alongside standard text. HTML tags are shown in light gray, making it easy to identify your code. You can also reposition HTML elements with drag-and-drop functionality. In the raw editor, the shortcodes for html look like this:

```
[:div class="myclass test" id="myid":]

This is text inside the div with an [:span class="myclass":]inline span element[:/span:] inside the div.

[:/div:]
```

Read all details about the HTML plugin on the [plugin page](https://plugins.typemill.net/html).

## FAQs, Tabs, Slideshows

The plugin also supports elements like FAQs, tabs, and slideshows. Simply add the appropriate HTML structure, classes, and IDs to ensure the CSS and JavaScript function properly on your site. The template plugin offers pre-made templates for these elements.

## Accordion Demo

The HTML plugin adds custom CSS and JavaScript to the frontend, enabling you to create an accordion-style interface like the following:

[:div id="accordion" :]

## How can I create a new question/answer?

[:div class="accontent" :]

To add a question, create a new headline. Wrap the corresponding answer in a `<div>` tag with the class `"accontent"`. Enclose the entire accordion in a `<div>` with the ID `"accordion"`.  

[:/div:]

## Where can I see the working accordion?

[:div class="accontent" :]

Publish the page and view it on the frontend to see the accordion element in action.  

[:/div:]

## What type of content can I use?

[:div class="accontent" :]

There are no limitations for answers—you can use any content type. However, for questions, only headline elements are allowed.  

[:/div:]

## Can I add multiple FAQ elements to one page?

[:div class="accontent" :]

Currently, you can only integrate one accordion element per page. Adding multiple accordion elements may cause them to malfunction.  

[:/div:]

[:/div:]

## Slideshow Demo

Slideshows can contain images or any other content elements.  

[:div id="slideshow":]

[:div class="slide active":]

![Screenshot](media/live/jongsun-lee-l84ef9qrgt0-unsplash2.webp){.center loading="lazy" width="820" height="547"}
*Photo by [Jongsun Lee](https://unsplash.com/@sarahleejs) Unsplash     *

[:/div:]

[:div class="slide":]

![](media/live/mathew-schwartz-jwj1kiux42c-unsplash.webp){.center loading="lazy" width="820" height="547"}
*Photo by [Mathew Schwartz](https://unsplash.com/@cadop) on Unsplash*

[:/div:]

[:div class="slide":]

![](media/live/hendrik-kuterman-ryxuv6g5asa-unsplash.webp){.center loading="lazy" width="820" height="547"}
*Photo by [Hendrik Kuterman](https://unsplash.com/@hendrik_martin) on Unsplash*

[:/div:]

[:div class="slide-nav":]

* [‹ previous](#prev-slide)  
* [next ›](#next-slide)

[:/div:]

[:/div:]

## Tab Demo

Tabs are useful for various types of content. In documentation, they are often used to switch between coding examples for different languages. 

[:div id="tabs":]

* [Usage](#tab1)  
* [Custom Style](#tab2)  
* [Limitations](#tab3)  

[:div id="tab1" class="tab-content":]

To create a new tab, simply add a new list item with an anchor linking to the tab ID. The tab content should be wrapped in a `<div>` with an `id` and the class `"tab-content"`.  

[:/div:]

[:div id="tab2" class="tab-content":]

If you want to customize the tab styles, use the custom CSS field in the theme settings.  

[:/div:]

[:div id="tab3" class="tab-content":]

You can only use one tab element per page. Adding multiple tab elements may cause them to malfunction. If you are using the Ebook plugin, keep in mind that tabs and other interactive elements do not work in formats like PDF or ePub.  

[:/div:]

