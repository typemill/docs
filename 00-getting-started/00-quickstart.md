#  Quickstart

Get started with Typemill quickly and easily! This guide will help you set up your first project in no time. Update.

## Install Typemill

If you already have a PHP-based website, installing Typemill should take less than five minutes. You can install Typemill manually, via GitHub, or with DockerHub. Just follow the [short installation guide](/getting-started/installation).

## Create Your First User

After installation, visit your website's URL. You’ll be redirected to a setup form to create your user account with a username, email, and password.

## Login

Once your account is created, you can log in to the system at the URL `/tm/login`.

## Configure the System

Upon your first login, you will be taken to the system settings. For a quick start, you can keep the default configurations or customize settings for:

* System
* Plugins
* Themes

## Create Content

Right after installation, you will have access to a demo website that showcases various features like hidden pages, restricted pages, and redirects. Log in to the author area (/tm/login) and navigate to "content" in the top navigation to edit all content:

* Use the interactive drag-and-drop feature.
* Organize your website with folders and pages.
* Write content using Markdown syntax.
* Switch between a visual editor and a raw Markdown editor.
* Edit page metadata in the meta tabs.
* Create sections for news or blog posts.

## Develop a Theme

Typemill offers a variety of themes, but you can also create your own using Twig and YAML. Refer to the theme developer guide and use the dev-theme as a starting point.

You can explore variables and functions for your theme and check out examples for the navigation, article lists, pagination, and image manipulation.

## Develop a Plugin

The Typemill plugin system utilizes the widely used Symfony Event Dispatcher along with Vue components for the admin interface. Create configuration forms with YAML, add new tabs to the content area, and introduce new items to the system navigation. But this is not all:

* Want to create a new user role? It's straightforward.
* Need to extend the editor? Check the math plugin as an example.
* Interested in integrating a newsletter? Shortcodes will help.
* Adding new routes and middleware? That is also manageable.

