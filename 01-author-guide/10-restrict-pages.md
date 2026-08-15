#  Restrict Pages and Folders

If simply [hiding a page](/author-guide/edit-meta-information) from the navigation isn’t sufficient, you can restrict access to individual pages, specific folders, or even your entire website to authenticated users, particular user roles, or designated usernames.

## Activate Access Management

To enable page restrictions, first activate access management in the system settings.

![Screenshot of the access management in Typemill](media/live/typemill-restrict-pages.webp){.center loading="lazy" width="820" height="514"}

You have several options:

* **Website restriction**: Restrict access to the entire website. Visitors who aren’t authenticated will be redirected to the login screen. This is especially handy when setting up your site live, allowing you to lift restrictions on launch day.
* **Page restriction**: Restrict access to individual pages. Once activated, you’ll find a roles and rights section in the meta tab of each page.
* **Hide Restricted Pages**: When enabled, restricted pages will be hidden from frontend navigation for users without sufficient permissions. Note that this may impact performance on sites with many pages.
* **Content break**: By default, restricted pages display only their title on the frontend. To hide the content instead, enable this option to insert a horizontal line.
* **Restriction notice**: If content is hidden using the content break, you can customize the message shown below the break using Markdown.

## Restrict Pages

After activating access management, new roles and rights fields become available in the meta tab of each page.

![Screenshot page restrictions](media/live/meta-page-restrictions.webp){.center loading="lazy" width="736" height="326"}

You can:

* Set a **minimum user role** required to view the page content on the frontend. See the user management and roles chapter for details.
* Specify a **comma-separated list of usernames** to grant exclusive access to certain users.

## Restrict Folders

You can also restrict access to specific base folders within user settings. Enter one or more folders, separated by commas, in the "Folder Access" field of the [user settings](/admin-guide/users), for example, "/folder1,/folder2". Authenticated users will then see only these folders in both the frontend and the author interface. If a user is not authenticated, all content remains visible on the frontend, but you can combine this with page-level restrictions to create any access scenario you need.

![Screenshot of folder access feature in the user settings of Typemill](media/live/typemill-account-restrictions-new.webp){.center loading="lazy" width="820" height="553"}

Keep in mind that folder restrictions apply only to the top-level navigation folders; nested (deep-level) folders cannot be restricted this way.

A common use case is multi-author sites, where each author has access to only their assigned chapter or folder in the editor. You can also combine folder restrictions with [multiple projects](/author-guide/multi-project) to build team-based knowledge bases with dedicated access, create multiple publications (e.g., training materials for different courses), or similar setups.

## Common Scenarios

* **Prepare for launch**: Restrict your entire website during setup and content preparation, then deactivate restrictions when you’re ready to go live.
* **Premium Content**: Use page restrictions and content breaks with a custom notice directing users to a registration page (register plugin). Only logged-in users can access the full content.
* **Restricted Content**: Restrict pages and hide them from navigation. For example, offer **training materials** for various courses, assigning different users access to specific course pages. You can share one account per course or list multiple users separated by commas. Combine this with the multi-project feature for dedicated course materials.
* **Internal Website**: For an internal company site (like a wiki), restrict the entire website to authenticated users and limit specific content to certain user roles. Use predefined roles and usernames or develop a plugin for company-specific roles.

## Read More

* [Hiding Pages ](/author-guide/edit-meta-information)
* [Multi-Project Website](/author-guide/multi-project)  
* [User Settings](/admin-guide/users)

