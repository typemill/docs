#  Troubleshooting Guide

Here you can find instructions to solve common problems and report errors so we can help fix them. You can also search for similar [issues on GitHub](https://github.com/typemill/typemill/issues) or in the [discussions on GitHub](https://github.com/typemill/typemill/discussions).

## Errors in Setup

Many reported errors during the setup process result from a misconfiguration of servers. If you are using an Apache server, make sure that:

* The `.htaccess` file is present.
* `mod_rewrite` is activated.
* Your Apache configuration is correct.
* Your permissions are set correctly.

In some environments (like **1und1**), you have to use an empty slash as the rewrite base. If you encounter any issues (e.g., receiving a 500 error from the server), please activate the following line in your `.htaccess` file and try again:

```
RewriteBase /
```

## Setup URL Returns a 404

If you receive a 404 error when you visit the setup URL `/tm/setup`, check if the `settings` folder is empty. If you find files or folders such as:

* `settings.yaml`
* `user` (folder)

Then delete both and revisit the page.

## Navigation Links Do Not Work After Switching the Domain

Typemill caches the navigation with absolute URLs, which causes the navigation to link to the old URL. To resolve this issue, go to the author interface, open the Kixote interface, and type `clear navigation` and `clear cache`. After that, the navigation will be automatically recreated with the correct base URL.

## Cannot Load External Scripts or External Content

Unlike many other CMS platforms, Typemill has implemented a Content Security Policy (CSP) that prevents the integration of external content. You have several options to allow external content to function:

* Integrate the external content or script using a plugin. With the plugin, you can add the URL to the CSP rules.
* In the admin interface, you can manually add URLs that should be allowed for your website. In the main navigation, go to `system` and open the `developers` tab.
* In the same tab, you can also disable the CSP feature completely for the website.

## Report Errors

If you cannot find a solution, please report an error on [GitHub](https://github.com/typemill/typemill/issues). Provide detailed information about the error by following these steps: 

### Enable Error Reporting

**If you have access to the admin area of Typemill**, go to the system settings in Typemill and switch to the tab `developers`. There you can activate the checkbox for "**Error Reporting**".

**If you do not have access to the admin area**, you can activate error reporting manually. Open the file `settings.yaml` in the `settings` folder and add the following line:

```yaml
displayErrorDetails: true
```

Do **NOT FORGET** to disable error reporting again after your tests, as detailed errors should never be displayed to visitors for security reasons.

### Enable Developer Tools in Browser

Enable the developer tools in your browser. The details depend on the browser you are using. The screenshots below are from Firefox.

With the developer tools open, you can recreate the error. In the developer tools, you will see detailed error messages in the "console" or "network" tabs. In the network tab, look for calls that are marked in red. Click on the red call, and in the window on the right side, click on the "response" tab. There, you should find a detailed error report that will help resolve the issue.

