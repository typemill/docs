# Render Markdown

Let's imagine that you want to create a sidebar box in your theme, and the user should be able to add text and use Markdown to format it, e.g. add a link. You can realize this with the `{{ markdown() }}` helper function in two steps:

In the first step you define a textarea for the sidebar widget in the YAML-file of your theme like this:

````
        sidebar:
          type: textarea
          label: Add some text for the sidebar with markdown
          placeholder: Add markdown here
````

Then you can render the Markdown content in your theme like this:

````
<div class="sidebar-box">
    {{ markdown(settings.myplugin.sidebar) }}
</div>
````

