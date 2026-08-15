#  Manage Users

Typemill provides intuitive and effective user management. You can create users, delete users, search for users, and assign different user roles with specific rights.

![Screenshot of the user management in Typemill](media/live/user-management.webp){.center loading="lazy" width="819" height="385"}

## User Roles and User Rights

Typemill offers several standard user roles with varying rights. If you have admin rights, you can change a user's role in the user management section:

* **Guest**: Can authenticate and view articles on the frontend but has no access to their account data.
* **Member**: Can view, update, and delete their account but cannot change their user role.
* **Contributor**: This role is currently not in use.
* **Author**: Can perform all actions that a member can do. Additionally, they can view all pages, create new pages, and update their own pages. They cannot publish, unpublish, or delete content.
* **Editor**: Can perform all actions that an author can do. Additionally, they can update, publish, unpublish, and delete all pages, even if they belong to other users.
* **Manager**: Has access to all administrative features, except for user management.
* **Administrator**: Can perform all actions and has access to all system configurations.

A developer can [change the existing roles](/plugin-developers/users) and create new roles.

## Guest Role

A guest user can only authenticate and view content pages on the frontend. They have no access to their own account data.

The guest role is useful if your website is non-public and you want to grant read access to authenticated users. You can also activate the option to log in with a link for guest accounts. This feature can be beneficial if you want to provide links for read access to your non-public website. Read the article about the [login link feature](/admin-guide/login-link) for more details.

## Member Role

A member user has read access to the frontend pages. Unlike a guest, a member can also change their account details and delete their account.

The member role is useful if you want to restrict your whole website or specific pages so that only authenticated users have full access to all articles. You can manage member accounts as an administrator or use the registration plugin, allowing members to register and manage their accounts without the assistance of an administrator. The registration plugin also offers an optional Gumroad license feature for selling access.

## Author Role

An author can see all pages in the editor area but can only create and update their own content; they cannot delete, publish, or unpublish anything. They do not have access to the settings area.

The author role is suitable for guest writers, as it prevents them from altering existing pages or publishing content that you have not yet reviewed.

## Editor Role

Editors can edit all pages from all users and can also delete, publish, and unpublish any pages. They do not have access to the settings area or administrative features.

This is a typical role for an editor or head of content, who can edit and publish all content on the website without any restrictions.

## Manager Role

The manager has access to all administrative features but does not have access rights to user management.

This administration role is suitable if you want to separate user management from other administrative functions. The management of user data can have special restrictions in companies due to data privacy compliance and GDPR.

## Administrator

Administrators can perform all actions, configure plugins, change themes, manage system settings, manage users, and change user roles. You should be very restrictive with this role.

