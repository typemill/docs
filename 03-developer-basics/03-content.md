#  Content

In Typemill, all content is stored as flat files. The text content is written in Markdown files (`.md`), while all metadata and page settings are stored in separate YAML files (`.yaml`). Media files like images, videos, or documents are stored in the `/media` folder and simply referenced from within the Markdown or YAML files.

## Folder and File Structure

Typemill organizes content in a folder and file structure—very similar to working with files on your local computer. You can create deeply nested folder structures to organize your content.

A special feature of Typemill: Folders can also have their own content, just like pages. This is done with an `index.md` for the content and an `index.yaml` for the metadata of that folder.

Here is an example of how your `/content` folder might look:

```
/content 
  index.md // Content of the homepage 
  index.yaml // Metadata of the homepage
  01-my-first-page.md // Content of /my-first-page 
  01-my-first-page.yaml // Metadata of /my-first-page
  /02-subfolder 
    index.md // Content of /subfolder 
    index.yaml // Metadata of /subfolder

/content
  index.md // markdown content of the homepage
  index.yaml // metadata of the homepage
  01-my-first-page.md // markdown content of /my-first-page
  01-my-first-page.yaml // metadata of /my-first-page
  /02-subfolder
    index.md // markdown of the /subfolder
    index.yaml // metadata of the /subfolder 
    01-my-subfolder-page.md // markdown of the /subfolder/my-subfolder-page
    01-sub-page.md         // Content of /subfolder/sub-page
    01-sub-page.yaml       // Metadata of /subfolder/sub-page
    01-my-subfolder-page.yaml // metadata of the /subfolder/my-subfolder-page
  03-another-page.md // Content of /another-page 
  03-another-page.yaml // Metadata of /another-page
```

## Pages and Posts

By default, all pages and folders in Typemill are sorted in a hierarchical order. To control the order, each file or folder is prefixed with a number like `01-`, `02-`, `03-`, and so on. Inside subfolders, the numbering starts again with `01-`.

You can easily change the order of pages and folders by moving them in the interactive navigation within the author interface of Typemill.

In addition to the manual sorting of pages, Typemill also supports "Posts." Posts are sorted automatically by date (newest first)—similar to a blog or news system. The format for posts is always `YYYYMMDDHHMM-title.md` (Year, Month, Day, Hour, Minute).

```
/02-folder-with-posts
  202502011834-my-newer-post.md
  202502011834-my-newer-post.yaml
  202501051523-my-older-post.md
  202501051523-my-older-post.md
```

To turn a folder with pages into a folder with posts, just use the radio selection in the meta-tab of a folder. It will automatically transform the files for you.

When a folder contains posts, a few things work differently:

* The posts are ordered automatically by their date—you cannot change their order with drag & drop.
* In the navigation of the author interface, posts are not listed directly. Instead, there is a separate list of posts shown below the content structure.
* To change the order of posts, simply edit the date of the post in the *Meta* tab of the post editor.
* Post-folders cannot contain subfolders—it is always a flat list of posts (just like blog posts in WordPress).

## Draft and Live

In Typemill, each page can have two different versions:

1. A *Live* version → published and visible on the website.
2. A *Draft* version → unpublished changes you are still working on.

The live version of a page is stored as a plain Markdown file (`.md`). You can easily read or edit it with any text editor. The draft version is stored as a `.txt` file in a special array-based format. This structure allows faster processing for the block editor but is not meant to be edited manually.

Example of a page with a live version and a draft version:

```
/content
  01-my-first-page.md 
  01-my-first-page.txt 
  01-my-first-page.yaml 
```

Example of a page that only exists as a draft and has not been published yet:

```
/content
  01-my-first-page.txt 
  01-my-first-page.yaml 
```

Example of a page that only exists as a live page (no draft changes):

```
/content
  01-my-first-page.md 
  01-my-first-page.yaml
```

When you publish your draft, the txt-version is transformed into Markdown and will overwrite the existing md-version.

## Updating Content Manually

If you want to update the content of Typemill manually (e.g., via FTP or with a technical workflow), then please read the chapter about [running Typemill without an admin](/admin-guide/use-without-admin).

