# Markdown Basics

Typemill is a Markdown CMS. Markdown is a simple text formatting language. By using a handful of special characters like `#` and `*`, Markdown allows for intuitive text formatting that can be learned in less than 10 minutes. Here’s a brief example of Markdown text:

````
# My first level headline

This is a paragraph, and now we create an unordered list:

- Item
- Another item
- A last item
````

Markdown has several advantages over traditional HTML editors. One key benefit is that tools like Typemill create portable and standardized Markdown files, which can be easily reused or processed with other tools. In contrast, many CMS store content as HTML in a database, which makes it harder to use the content outside of that system. Markdown has also become a widely adopted standard for content creation in tools like Slack, Reddit, and GitHub.

## Typemill Markdown Editors

Typemill features two distinct Markdown editors:

* [Visual Markdown Editor](/author-guide/visual-markdown-editor): This block editor provides a WYSIWYG mode (what you see is what you get). This mode is good for easy and intuitive content creation of smaller content blocks. 
* [Raw Markdown Editor](/author-guide/raw-markdown-editor): An editor that highlights raw Markdown syntax. This mode is good for copying, pasting, or reorganizing larger sections of your article.

You can easily switch between both editors with a button in the sticky publish bar and configure a default editor in the system settings.

## Markdown Compatibility

Markdown is a standarized syntax but with different flavours. Typemill supports all elements defined in the original [Markdown specification](https://daringfireball.net/projects/markdown/syntax) by John Gruber and most elements defined in the [Markdown extra specification](https://michelf.ca/projects/php-markdown/extra/). For an example with all markdown elements featured in Typemill, please visit the [Markdown reference page](/author-guide/markdown-demo). You can also check the test page of the [Parsedown library](https://parsedown.org/tests/) that is used in Typemill.

## Standard Markdown

List of the basic Markdown syntax: 

| Markdown | Result |
| -------- | ------- |
| `# Heading 1` to `###### Heading 6` | Headings (levels 1 to 6) |
| `simple text` | Paragraph |
| `*italic*` or `_italic_` | Italic text |
| `**bold**` or `__bold__` | Bold text |
| `> blockquote` | Blockquote |
| `- item`, `* item`, `+ item` | Unordered list |
| `1. item` | Ordered list |
| `` `inline code` `` | Inline code |
| Indented with 4 spaces or fenced with triple backticks | Code block |
| `---`, `***`, `___` | Horizontal rule |
| `[link text](http://example.com)` | Link |
| `![alt text](http://example.com/image.jpg)` | Image |

## Markdown Extra

List of the Markdown extra syntax that is widely adopted:

| Extra Element        | Description |
| ------------------- | ----------- |
| Tables              | Support for tables with optional alignment of columns. |
| Fenced Code Block with Language | Fenced code blocks with optional language identifier for syntax highlighting. |
| Definition Lists    | Support for definition lists using `<dl>`, `<dt>`, and `<dd>`. |
| Header IDs          | Add custom IDs to headings (e.g., `## Title {#custom-id}`). |
| Attribute Lists     | Add attributes (class, id, style) to block elements (e.g., `{.class #id}`). |
| Footnotes           | Create footnotes using `[^1]` syntax with definitions at the end of the text. |

## Typemill Specific Options

Typemill supports some more elements that are not standard, but often used in other tools: 

| Syntax     | Result                      |
| ---------- | --------------------------- |
| `[TOC]`    | Inserts a table of contents based on the headings in the page. |
| `! Info` | Creates an info notice block. |
| `!! Info 2` | Creates a warning notice block. |
| `!!! Info 3` | Creates a success notice block. |
| `!!!! Info 4` | Creates an error notice block. |

## Restrictions for HTML

Please note that Typemill does not allow the use of HTML in Markdown for security reasons. If you want to add HTML, you can use the HTML plugin, which lets you insert certain HTML tags securely via shortcodes.

