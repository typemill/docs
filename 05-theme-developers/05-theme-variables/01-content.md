# content

The Twig-variable `{{ content }}` holds the whole content of your Markdown file in HTML.

## Example Usage

    {{ content }}

You can use Twig filters, but the usecases are limited. For example, you can strip out all tags to get pure text output:

```twig
{{ content|striptags }}
```



You could also slice content or use other manipulations, but in general it makes more sense to create plugins to manipulate content or to add extra content (e.g. with shortcodes).

