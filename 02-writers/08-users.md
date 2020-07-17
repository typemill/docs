# User Roles and User Rights

Typemill provides some standard user-roles with different rights. If you have admin-rights, you can change the role for a user in the user-management: 

* **Member**: Can view, update and delete his account but cannot change his user-role.
* **Author**: Can do everything a member can do. Additionally can view all pages, create new pages and update his own pages. He cannot publish, unpublish or delete content.
* **Editor**: Can do everything an author can do. Additionally can update, publish, unpublish and delete all pages, even if they are from other users.
* **Administrator**: Can do everything and has access to all system configurations.

A developer can change the existing roles and create new roles.

## Usecases for Members

If you want to restrict your website from public access, then you can set a password with htpasswd (similar to htaccess) and share that single password to all users, who should have access to the website. Another way is to restrict the whole website to members in the system settings and create individual accounts for each member. 

Other usecases are in work right now and will be implemented with plugins. One example is a member-plugin, where members can register with a registration form and haver access to certain content and functions, for example to member content or to downloads. There are also plans to introduce payed member roles for pro-content in future.

## Usecases for Authors

An author can only create and update their own content, but they cannot delete, publish or unpublish anything. This is a good role for guest writers for example, so they cannot mess up existing pages or publish content, that you did not check yet.

## Usecases for Editors

Editors can edit all pages from all users and also delete, publish and unpublish all pages. This is a typical role for an ... editor. Or a head of content. Or a chief content executive. Or a master of content.

## Administrator

Administrators can do everything, configure plugins, change themes, configure system settings, manage users and change user roles. You should be very restrictive with that role, of course.

