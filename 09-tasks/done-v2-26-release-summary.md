# Typemill 2.26.0 Release Notes

Release 2.26.0 focuses on security hardening, PHP 8.2–8.5 compatibility, and quality-of-life improvements for the bundled demo plugin and core models. The release covers the changes committed to the `v2.x` branch after the `2.25.0` tag (`a35b7f9`) up to the current HEAD (`c41bbb1`).

---

## 1. Security Fixes

### 1.1 Authorization Bypass in Media Downloads

**File:** `system/typemill/Controllers/ControllerWebDownload.php`

The role-restricted file download route was vulnerable to path-equivalence bypasses. An unauthenticated attacker could request variants such as `/media/files/./secret.pdf`, `/media/files//secret.pdf`, or `/media/files/%2e/secret.pdf` and bypass the restriction check while still resolving to the same file.

Changes:

- Added a `normalizeFilename()` helper that URL-decodes, collapses `//` segments, removes `./` segments, and rejects `..` and NUL bytes.
- The restriction lookup and file read now use the same normalized path.
- Added `realpath()` confinement: the resolved file must be inside the resolved `media/files` directory.
- Bypass attempts now return `404` instead of the restricted file.

**Task reference:** `ai/tasks/done-v2-26-security-bypass-media.md`

### 1.2 Stored XSS via Hero Image Alt Text in Cyanine Theme

**Files:** `themes/cyanine/*.twig`

The Cyanine theme rendered the `heroimagealt` meta field and other meta fields raw in HTML attributes. An author could store a payload such as `"><img src=x onerror=alert(document.domain)>` in the Hero Image Alt Text field and execute JavaScript on the public folder page.

Changes:

- Added `|e('html_attr')` escaping to all `heroimagealt` outputs.
- Added `|e` escaping to `page.meta.title`, `post.meta.title`, `post.meta.author`, `post.meta.owner`, `page.meta.shorttitle`, and description fields.
- `blog.twig` now uses `post.meta.heroimagealt` for the alt text instead of `post.meta.heroimage`.
- Cyanine theme version bumped to `2.4.2`.

**Task reference:** `ai/tasks/done-v2-26-security-css-hero-image.md`

### 1.3 Stored XSS via Dangerous URI Schemes in Markdown Links

**File:** `system/typemill/Extensions/ParsedownExtension.php`

Markdown links using dangerous schemes (`javascript:`, `vbscript:`, `data:text/html`, etc.) were rendered as clickable links. The existing sanitizer only covered the direct inline-link branch and did not protect reference-style links or images.

Changes:

- Added a protected `sanitizeUrl()` helper that reuses Parsedown's `safeLinksWhitelist` plus the user-defined `urlschemes` setting.
- The helper is applied to both direct inline links and reference-style links/images.
- Allowed schemes (e.g., `http`, `https`, `mailto`, `ftp`, `tel`, `data:image/png;base64`) and relative links remain intact.

**Task reference:** `ai/tasks/done-v2-26-security-xss-inline-link.md`

---

## 2. PHP Compatibility

### 2.1 PHP 8.2 – 8.5 Support

**Files:** `composer.json`, `system/typemill/Plugin.php`, `system/typemill/Controllers/Controller.php`, `system/typemill/Models/ApiCalls.php`, `system/typemill/Models/Media.php`, `Dockerfile`

Typemill now declares compatibility with PHP 8.2, 8.3, 8.4, and 8.5. PHP 8.1 is no longer supported.

Changes:

- `composer.json` updated to `^8.2 || ^8.3 || ^8.4 || ^8.5` and platform pinned to `8.2.0`.
- Updated Slim, Twig, PHP-DI, Symfony YAML/Event Dispatcher, Laminas ACL, and PHPMailer to versions compatible with PHP 8.2+.
- Fixed implicit nullable parameter deprecations in `Plugin::generateStaticAsset()` and `Plugin::getStaticAssetPath()` (`string $namespace = null` → `?string $namespace = null`).
- Replaced `null` as the first select-box option with an empty string in `Controller::getFormFields()` to avoid PHP 8.5 deprecation warnings.
- Removed deprecated `curl_close()` calls from `ApiCalls`.
- Removed deprecated `imagedestroy()` calls from `Media` (GD handles cleanup automatically in modern PHP).
- `Dockerfile` updated to `php:8.5-apache`.

**Task reference:** `ai/tasks/done-v2-26-updat-php-version.md`

---

## 3. Core Model Improvements

### 3.1 Folder and Navigation Prefix Handling

**Files:** `system/typemill/Models/Folder.php`, `system/typemill/Models/Navigation.php`

The folder parser previously treated any dash-separated prefix as an ordering prefix. This caused files like `theme-development.md` to lose the `theme` segment and produced incorrect slugs.

Changes:

- Added `extractOrderAndSlug()` and `isDateOrder()` helpers.
- Only plain numeric prefixes or ISO date prefixes (`2023-01-01`, `20230101`) are now stripped as ordering prefixes.
- Other dash-separated segments are preserved in the item name and slug.
- `Navigation::renameItem()` correctly rebuilds filenames with or without an order prefix.

### 3.2 Storage Model Robustness

**File:** `system/typemill/Models/Storage.php`

Files without an extension (e.g., `README`) no longer cause a PHP warning when `pathinfo()` does not set the `extension` key.

### 3.3 Mail Model Enhancements

**File:** `system/typemill/Models/Mail.php`

- Added optional `replyTo`, `replyToName`, and `altBody` parameters to `Mail::send()`.
- Reply-to addresses can be supplied per call instead of relying solely on the global configuration.

### 3.4 User Profile Image Handling

**File:** `system/typemill/Models/Media.php` (and related controllers)

Fixed broken image handling for user profile images.

### 3.5 Manual Date Handling in Meta API

**File:** `system/typemill/Controllers/ControllerApiAuthorMeta.php`

Added a type guard before checking `strlen($item->order)` to prevent errors when the order value is not a string.

---

## 4. Demo Plugin Improvements

**Files:** `plugins/demo/demo.php`, `plugins/demo/demo.yaml`, `plugins/demo/css/demo.css`

The bundled demo plugin was expanded to better illustrate Typemill plugin capabilities:

- Added a public frontend page (`/demo`) with a generated form and validation.
- Added a form processor route (`/demoprocessor`) with CSRF/session segments support.
- Added a configurable page widget (`showwidget`, `widgettitle`, `widgetmessage`) that injects a sanitized HTML widget into public pages.
- Added demo assets for the editor/blox integration (`editordemo.js`, `demobloxconfig.js`, `demobloxcomponent.js`, `widgetdemo.js`).
- Added success/error messages and label fields for the public demo form.
- Reorganized form fields with `lg:w-half` CSS for two-column layout.

**Task references:** `ai/tasks/done-v2-26-todo-*` (general v2.26 plugin work)

---

## 5. Frontend / Admin UI Updates

### 5.1 JavaScript Library Updates

**Files:** `system/typemill/author/js/*.js`

- Updated Vue.js from `3.5.35` to `3.5.41` (stable bug fixes and hydration improvements).
- Updated Axios and Sortable.js to current patch versions.

### 5.2 System and Plugin Lists

**File:** `system/typemill/Static/Plugins.php`

- Added `askthedocs` to the premium plugin license list.
- Fixed the typo in the newsletter plugin class path (`newslettter` → `newsletter`).

### 5.3 Debug Output

**File:** `system/typemill/system.php`

When `displayErrorDetails` is enabled, the core now also sets `display_startup_errors` and `error_reporting(E_ALL)` for more complete diagnostics on PHP 8.x.

---

## 6. Packaging and Development

- `.gitignore` reorganized for clarity; added `zips`, `cypress`, `node_modules`, and `ai/tasks` entries.
- `package-lock.json` updated for Tailwind CSS 4.3.3 and related dependencies.

---

## 7. Tasks Tracked for v2.26 but Not in the Current Branch Snapshot

The following task files in `ai/tasks/` document additional work that was scoped for the 2.26 cycle but is **not present in the current `v2.x` branch snapshot** (`a35b7f9..c41bbb1`):

- `done-v2-26-todo-gitsync-plugin.md` — GitSync plugin MVP implementation plan and review.
- `done-v2-26-todo-gitsync-refactor.md` — UI refactor, GitLab support, selective sync, and loading feedback.
- `done-v2-26-todo-gitsync-improvements.md` — Metadata restructuring and repository status endpoint.
- `done-v2-26-todo-gitsync-docs.md` — User-facing GitSync documentation.
- `done-v2-26-todo-ebookplugin-onhtmlexportloaded.md` — Ebook plugin integration with the `OnExportHtmlLoaded` event.

These files contain implementation plans and documentation drafts. They may live in a separate feature branch or be targeted for a later 2.26.x / 2.27 release.

---

## 8. Upgrade Notes

- The minimum PHP version is now **8.2**.
- Run `composer install` after pulling 2.26.0 to update dependencies.
- If you are using the Cyanine theme, republish the cache or clear `/cache` to ensure the updated Twig templates are picked up.
- Review any custom themes that render meta fields in HTML attributes and add `|e` or `|e('html_attr')` escaping where appropriate.

---

**Release range:** `a35b7f9` (Finish Version 2.25.0) → `c41bbb1` (update gitignore) on branch `v2.x`.
