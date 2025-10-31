#  Use the Google Sitemap

A sitemap is an XML file that lists the web pages of your website to inform Google and other search engines about the organization of your website content. Search engine web crawlers like Googlebot read this file to crawl your site more intelligently.

If you are looking for an SEO solution, then check out the Typemill bundle for [SEO websites](/solutions/seo-website). It ships with several plugins that help you create high-ranking content.

## A Sitemap for Typemill

Typemill creates a sitemap out of the box and stores it in the cache folder. You can find the URL for the sitemap in the system settings. Simply add the URL to the Google Search Console or other search engines, and you are done.

If you face any problems with submitting the sitemap, then add the urls `https://www.google.com` and `https://www.googlebot.com` to the whitelist for CSP. [Read more here](/admin-guide/developer-tab).

## Managing Your Typemill Sitemap

* All pages of your website are included in the sitemap.
* You can exclude individual pages from the sitemap manually. Navigate to the meta tab of a page, scroll down to "visibility," and activate the checkbox "noindex." This feature is particularly useful for pages that you do not wish to appear in search results.
* The sitemap is dynamically updated in various scenarios, ensuring that your website's structure is accurately reflected. It will be recreated if you publish a page, unpublish a page, move a page to another folder, or delete a page.

## Disable the Sitemap

The generation of the sitemap can slow down some actions like publishing or unpublishing pages. To improve performance slightly, you can disable the sitemap in the system settings (starting with version 2.3.4). This makes sense in several scenarios:

* Your website is not publicly accessible. In this case, a sitemap is not needed.
* You do not work with the Google Search Console. 
* Your website is very large, with several thousand pages.

