#  Use Typemill Without the Admin Interface

Sometimes, you may want to run your Typemill website without relying on the admin interface or author tools. Typemill offers several ways to manage your site without using the interface directly.

## Navigation Cache

As a flat-file CMS, Typemill allows you to manually upload content files into the `content` folder. However, Typemill maintains a cache of the navigation structure, so manually added files won’t appear in the navigation until the cache is cleared and rebuilt. When using the interface, this process happens automatically whenever you create, delete, or move a page. Alternatively, you can use the Kixote admin tool to clear the cache and navigation via command. Without the admin interface, you have the following three options.

## Option 1: Auto-Refresh Navigation

Starting with Typemill version 2.18.0, there’s a new option under the developer tab in system settings called "[Auto-Refresh Navigation](/admin-guide/use-without-admin)." Enabling this option causes the navigation cache to be cleared and rebuilt automatically at set intervals, ensuring that page changes (additions, deletions, moves) are reflected in the navigation. By default, this refresh occurs every 10 minutes, but you can customize the interval in the developer settings.

## Option 2: Delete Navigation via API

If you manage your content updates programmatically — such as moving files from a repository via a job — you can clear the cache and navigation using API calls. Although these API endpoints are primarily intended for internal use and are not officially documented, you can access them externally with a user having at least the "manager" role. The relevant endpoints are:

* `/api/v2/cache`
* `/api/v2/clearnavigation`

For more details on how to use the API, read the relevant [chapter](/api).

## Option 3: Delete Cache Files Manually

If you prefer not to use the API, you can clear the cache by deleting files directly (for example, with a bash script). You should remove:

* All files in the `data/navigation` folder.
* The `sitemap.xml` file in the `cache` folder.

You may also delete other cache files (such as the Twig cache, if enabled), but **do not delete any CSS files**, as they contain your custom theme styles.

## Naming Conventions and Performance Considerations

Keep in mind that regenerating the navigation can be slow on large websites, so avoid clearing the navigation cache unless necessary. Also, when adding content files manually, follow the [naming conventions for content files](/developer-basics/content). YAML metadata files for pages will be created automatically if they don’t already exist.

