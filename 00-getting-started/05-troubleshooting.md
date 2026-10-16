#   Troubleshooting Guide

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

![Screenshot of the network tab in the developer tools of firefox](media/live/error-reporting-firefox.webp){.center loading="lazy" width="820" height="489"}

With the developer tools open, you can recreate the error. In the developer tools, you will see detailed error messages in the "console" or "network" tabs. In the network tab, look for calls that are marked in red. Click on the red call, and in the window on the right side, click on the "response" tab. There, you should find a detailed error report that will help resolve the issue.

### Step-by-Step: Viewing Error Messages in Google Chrome

If you are using Google Chrome and are not very familiar with developer tools, follow these simple steps to find and copy the error message:

1. **Open the website** where the error occurs in Google Chrome.
2. **Open the Developer Tools.** You can do this in one of the following ways:
   * Right-click anywhere on the page and select **"Inspect"** from the menu, or
   * Press **F12** on your keyboard, or
   * Press **Ctrl + Shift + I** (Windows/Linux) or **Cmd + Option + I** (Mac).
3. A new panel will open, usually on the right or bottom of your browser window. This is the **Developer Tools** panel.
4. At the top of this panel, you will see several tabs, such as "Elements", "Console", "Sources", and **"Network"**. Click on the **"Network"** tab.
5. **Reload the page** to make sure the network activity is recorded. You can do this by pressing **F5** or clicking the reload button in your browser.
6. **Recreate the error.** Perform the action that caused the problem (for example, click the button or link that triggered the error).
7. Look through the list of requests in the Network tab. Requests that failed are usually shown in **red**, and the "Status" column will often show a number like `404` or `500`.
8. **Click on the red (failed) request.** A new panel will open on the right side (or below) showing more details.
9. In this new panel, click on the **"Response"** tab (sometimes also called "Preview" or "Response Preview").
10. You should now see the detailed error message. It may look like plain text or code. **Select all the text and copy it** (Ctrl + C on Windows/Linux, Cmd + C on Mac).
11. **Paste this error message** into your bug report on [GitHub](https://github.com/typemill/typemill/issues), along with a short description of what you were doing when the error occurred.

**Tip:** If you are unsure which request is the correct one, look for the request that matches the timing of your action, or the one with the red color and an error status code (like 404, 500, or 403).

