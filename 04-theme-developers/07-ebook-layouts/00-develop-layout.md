# Develop a Layout

The eBook plugin generates an eBook preview based on a specific eBook layout. As a developer you can code your own layout and it works pretty similar to [Typemill themes](/theme-developers). All you need are some simple YAML-definitions and a Twig-template with HTML, CSS and optionally some JavaScript.

You can find all eBook-layouts in the folder "booklayouts" of the eBook-plugin. Each layout has only a handfull of standard files:

* **config.yaml**: Contains some basic information and form-definitions with yaml syntax.
* **cover.png**: A preview image for your layout.
* **index.twig**: The twig template for your eBook.
* **style.css**: The style for your eBook.

That's it. You can add more stuff, of course, for example a folder with your own fonts or some tricky JavaScript. But you don't need to with your own project. Let's go into some details for each file.

