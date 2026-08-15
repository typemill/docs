#  Edit Page Meta Information

You can edit detailed meta-information by switching to the Meta tab at the top of each content page. Plugins can extend this functionality by adding fields to the existing Meta tab or creating new meta-tabs with additional data or features.

![Screenshot of the meta settings in Typemill](media/live/edit-page-meta.webp){.center loading="lazy" width="821" height="473"}

## Slug and Navigation

Here you can change the slug and the navigation title of the page. The slug is the page-specific segment in the URL, so in a URL like `https://typemill.net/author-guide`, the `author-guide` is the slug.

![Screenshot of the slug and navigation field in the meta-tab of a Typemill page](media/live/meta-page-slug-navigation.webp){.center loading="lazy" width="773" height="199"}

! **Be Careful**
! 
! Changing the slug will alter the URL. In the editor view, the page will immediately reload with the new URL. Be aware that unsaved changes in the meta-tab will be lost. Also, consider the effects for search engines and bookmarks if the URL of a page changes.

## Meta Title and Description

Here you can change the meta title and the meta description. Check out the [SEO plugin](https://plugins.typemill.net/seo) if you need more features.

![Screenshot of the meta-title and meta-description fields in a Typemill page.](media/live/meta-page-title-description.webp){.center loading="lazy" width="757" height="448"}

## Hero Image

The hero image will be used when pages are shared on social media platforms. Depending on the theme that you use, the hero image can also appear in the frontend, e.g., as a header image in a list of posts or as a stage image.

![Screenshot of the hero image input in the meta-tab of a Typemill page](media/live/meta-page-hero-image.webp){.center loading="lazy" width="763" height="527"}

## Author and Owner

Here you can edit the owner of the page and the author name for the frontend.

* **Owner**: The owner is the username associated with the page. Initially, the page creator is assigned as the owner. The owner has edit permissions for the page. However, certain user roles, such as editor or admin, always have unrestricted edit rights.
* **Author**: The author name can appear in the frontend, depending on your theme. Initially, the author name is prefilled with the name of the user who created the page.

![Screenshot of the author feature in the meta-tab of a Typemill page](media/live/meta-page-authors.webp){.center loading="lazy" width="736" height="326"}

## Access Rights

You can restrict access to the page in the frontend. The fieldset for access rights will only appear if the page restrictions are activated. You can activate them in the system settings under the tab "access."

* **User role**: You can select a minimum user role that should have access to the page.
* **Usernames**: You can add a list of usernames separated by commas that should have access to the page.

![Screenshot of the access feature in the meta tab of Typemill](media/live/meta-page-restrictions.webp){.center loading="lazy" width="736" height="326"}

## Date

Here you can see and edit the date of creation, the date of last publication (last modified), and a manual date. The manual date is used in the frontend if set. The last modified date is used as a fallback.

![Screenshot of the date settings in the meta-tab of Typemill pages](media/live/meta-page-date.webp){.center loading="lazy" width="736" height="426"}

## Reference

You can create different references for a page, such as a redirect 301 or 302 to another page. The options are:

* **Permanent Redirect (301)**: Redirect visitors to a new page permanently. This feature is also useful in SEO.
* **Temporary Redirect (302)**: Temporarily redirect visitors to another URL.
* **Copy**: Copy page content to another page. Useful in many situations, but avoid this feature for public sites due to duplicate content penalties.
* **External Link**: Add navigation items leading directly to external URLs.

![The reference features in the meta tab of a Typemill page.](media/live/meta-page-reference.webp){.center loading="lazy" width="736" height="410"}

## Visibility

You can hide a page from the navigation and exclude it from the index with a noindex tag. If you hide the page from the navigation, then the page is still accessible to the public, but it will not be linked in the navigation or in any paging or lists. If you activate the noindex feature, the page will also be excluded from the automatically generated Google sitemap.

![The visibility feature in the meta-tab of a Typemill page](media/live/meta-page-visibility.webp){.center loading="lazy" width="736" height="258"}

## Folder

The fieldset for folders is only visible in meta-tabs for folders. Here you can change how the containing pages are listed:

* Pages sorted in navigation with drag & drop
* Posts sorted by publish date for news or blogs

Some themes also support a glossary view for folders. For details, read the chapter about [creating posts](/author-guide/create-post) and [creating a glossary](/author-guide/create-glossary).

![Screenshot of the folder feature in the meta-tab of a Typemill page](media/live/meta-page-folder.webp){.center loading="lazy" width="736" height="296}

