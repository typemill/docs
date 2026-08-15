#  Security Measures in Typemill

We have implemented numerous measures to ensure that Typemill remains as secure as possible. If you identify a potential security vulnerability within Typemill, please report it to us via email at security@typemill.net. We kindly ask you to provide as much detail about the issue as possible to help us address it efficiently.

## Security Configurations for Authors and Admins

Several configurations can be used to enhance your website's security.

### Use HTTPS

Always run your Typemill website with HTTPS. Some features might not work if you run Typemill on HTTP only.

### Login Verification

We highly encourage activating the [login verification feature](/admin-guide/login-verification) released in Typemill version 2.1.0. This feature sends a 5-digit verification code to your email to verify and complete your login attempt, providing a simple and effective measure against password theft and account hacking.

### Captcha Verification

You can activate captcha verification for all authentication forms in the security tab of the system settings. Additionally, if a plugin uses public forms, you can also activate captcha verification for those forms.

### Security Log

You can activate a security log in the security tab of the system settings. Typemill will log suspicious actions like incorrect login attempts with the time and IP address. You can view and delete the security log in the Kixote interface (type 'help' for a list of commands). Ensure this complies with the privacy legislation of your country.

### Password Recovery

[Password recovery](/admin-guide/forgot-password) is inactive by default due to additional security implications. If you need password recovery, you can activate it. Please follow the advice for better password security.

### SVG Uploads

By default, uploading SVG files into the media library is disabled because SVG consists of code that can contain malicious elements. You can activate SVG uploads in the media tab of the system settings. All SVG files will be sanitized with a set of whitelisted tags. Some SVG files with advanced features might not work after sanitization.

## Security Configurations for Admins and Developers

### Error Reporting

Error reporting is turned off by default to prevent error reports from being visible on public pages. You can turn on error reporting in the developer settings of the system settings for bug analysis but should turn it off immediately after finishing your analysis.

### Content Security Policy (CSP)

Typemill adds a content security policy (CSP) to all HTML pages. This means you cannot include content from other domains unless these domains are whitelisted. Whitelisting is usually done with plugins and themes. If you need to whitelist a domain manually, you can do so in the developer tab of the system settings. If you encounter any significant problems with the content security policy, you can also disable the CSP headers completely in the developer tab.

### API restrictions

All API endpoints in Typemill are secured using Basic Authentication. Typemill does not support CORS headers, meaning the API cannot be accessed directly from the browser (e.g., using JavaScript). Only server-side calls are allowed. Additionally, you can restrict API access based on IP addresses or hostnames, which can be configured in the Developer tab of the system settings.

### Custom Validation for Form Validation

If you code a theme or plugin, you can define configuration forms or public forms in the YAML configuration file. In addition to system-side input validation, you can add [custom validation](/forms/field-overview) rules like max and min or a regular expression to validate the input.

### Custom Headers

If necessary, you can disable the headers sent by Typemill and use your own. Disable the headers in the developer tab of the system settings. Be sure you understand the implications.

## Default Security Measures of Typemill

### Markdown Restrictions

Typemill does not allow any pure HTML or code input into the markdown editor. We use the Parsedown library in safe mode.

### Input Validation

All input in the editor and all forms are automatically validated with standard validation rules. We use the Valitron library for this. If you define custom forms in YAML, standard input validation is applied automatically, but you can still add custom validation with regular expressions for more specific cases.

### Honeypot and Captcha for Public Forms

All public forms receive simple honeypot validation by default. Additionally, all public forms have the option to activate captcha by default.

### File Uploads

All uploaded files are checked against their MIME type and the corresponding extension. Code files cannot be uploaded.

### SVG Uploads

If SVG upload is activated in the system settings, then all SVG files will be sanitized before being stored in the media library.

### Security Headers

All HTML pages are delivered with several security headers, including CSP headers for content security policy. You can whitelist domains for the content security headers in plugins, themes, and the admin interface, or you can disable CSP headers on pages in your plugin.

### CORS Headers

All API endpoints in Typemill automatically include CORS headers. You can whitelist domains to access your API endpoints with a plugin or in the admin interface.

### HTACCESS

The .htaccess file contains several security settings and blocks direct access to most file types, the root folder, and most other folders. It also contains an optional rule to redirect HTTP to HTTPS.

### Authentication and Authorization

All administrative pages and API endpoints require authentication with session cookies (website) or an API key in the authorization header (API endpoints). There is also an authorization check that inspects the role and rights of the current user.

### Basic Auth Credentials

Basic Auth credentials are removed from the URI object at the start of the application. This prevents credentials from being tracked in tracking software or exposed to the public.

