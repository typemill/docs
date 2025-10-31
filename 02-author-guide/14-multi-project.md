# Multi-Project: Create Multiple Projects with One Instance

By default, Typemill creates a website with a single navigation containing all pages.  With the multi-project feature, you can split your content into separate projects, each with its own navigation.

![Screenshot Multiproject Selection in the Author Interface](media/live/typemill-multiproject-edited.webp){.center loading="lazy" width="819" height="408"}

In the author area, admins can switch between projects using a dropdown menu above the navigation. On the frontend, you can either enable a project switcher or implement a theme-specific solution.

## Use Cases

Multi-project setups are especially useful for:

* multiple products  
* different book projects  
* separate versions  
* teams with independent content  
* different author groups  
* target-group–specific content  
* separate topics within one site

## Improve Performance for Large Websites

A multi-project setup is especially recommended for **websites with more than 2,000 pages**. Typemill only loads the navigation of the active project instead of the entire site structure. By splitting your content into several projects, you can manage websites with many thousands of pages without running into performance issues.

## Setup

You can configure projects in the system settings under the [projects tab](/admin-guide/project-tab):

1. Go to **System Settings**  
2. Open the **Projects** tab  
3. Define an **ID** (identifier) and a **Name** (label) for your base website  
4. Add IDs and Names for each sub-project  

The **ID** is used as slug for the subfolder. For example, a project with the ID `myproject` will be available at `https://mywebsite.org/myproject`. The **Name** (label) is shown in the dropdown menu.

The **base website** does not get an extra slug and remains available under the main domain. Its ID is only used internally for selection. You can add new projects at any time without changing the existing URLs of your base website.

## Create Content

Once a sub-project is created, a dropdown menu in the author area lets you switch between the base website and sub-projects.  
Each sub-project starts empty. You can manage content just like in the base website: create folders, add pages, and organize navigation.

## How It Works Internally

Each project corresponds to its own subfolder structure:

- **Content folder:** For each project, a subfolder is created with the pattern `_projectid`.  
- **Navigation folder (in `/data/navigation`):** Each project has its own navigation cache in a subfolder with the same pattern `_projectid`.  

This way, each project remains fully separated while sharing the same Typemill installation.

## Limitations

The multi-project feature is **not** a multi-instance feature. All sub-projects share the same configuration as the base website.  

This means you cannot assign different plugins, themes, or settings to individual sub-projects. The feature is only for separating content into distinct projects within one website. 

If you need multiple websites with different configurations, you must install Typemill separately for each one.

