#   Login Link

The **Login with Link** feature in Typemill allows guest users to log in via a special access link instead of using traditional authentication methods. This is similar to the public share links that services like Google Docs provide. It can be useful for linking users from a software environment to restricted frontend pages of a Typemill-based documentation system.

## Enabling Login with Link

To activate the **Login with Link** feature, follow these steps:

1. Navigate to **System Settings** (`/tm/system`).
2. Go to the **Developer** tab.
3. Enable the option **Login with Link**.

![Screenshot of the developer settings for the login link](media/live/login-link-developer-settings.webp){.center loading="lazy" width="820" height="378"}

If you integrate login links into a SaaS environment, consider restricting access to specific IP addresses in the developer settings under **Trusted IPs for the login-link-referrer (comma separated)**.

## Configuring Link Access for a Guest User

After enabling **Login with Link** in the system settings, you can configure individual guest users to use this feature:

1. Go to **User Management** (`/tm/users`).
2. Select a user with the role "guest".
3. In the **User Profile**, check the option **Link Access**.

![Screenshot login link](media/live/login-link-account-settings.webp){.center loading="lazy" width="820" height="446"}

**Important:** This feature is available only for users with the **Guest** role.

* A **Guest** user has no permissions beyond accessing frontend pages.
* Guests **cannot** modify content, manage settings, or access their own account settings.

## Create the Login Link

The login link feature works with simple GET parameters for the username and password, like this:

```
/tm/loginlink?username=yourusername&password=yourpassword
```

## Security Considerations

If you use this feature to control access to non-public documentation within a software ecosystem, consider the low security of this feature. A login link is similar to a public share link provided by Google and other services. Use whitelisting to restrict access to specific IP addresses.

## Summary

The **Login with Link** feature provides a convenient way for guest users to access protected frontend pages in Typemill. However, it should be implemented with caution due to its lower security level. Use IP restrictions and limit access only to necessary users to enhance security.

