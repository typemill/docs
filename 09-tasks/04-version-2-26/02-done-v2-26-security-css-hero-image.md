tored Cross-Site Scripting via Hero Image Alt Text in
TYPEMILL 2.25.0 Cyanine Theme
Researcher: Md. Moniruzzaman Prodhan
Website: https://nomanprodhan.com
GitHub: https://github.com/NomanProdhan
LinkedIn: https://www.linkedin.com/in/nomanprodhan/
Date: 31 July 2026
Summary
TYPEMILL 2.25.0 is affected by a stored cross-site scripting (XSS) vulnerability in the Hero Image Alt Text
field of the bundled Cyanine theme. An authenticated user with the Author role can store a malicious
payload in this field. Once an administrator publishes the folder and page, the Author can subsequently
modify the page metadata and inject the payload without requiring another administrative review or
publication.
The payload executes when a visitor opens the parent folder page while hero images are enabled for post
listings in the Cyanine theme. The JavaScript runs in the context of the TYPEMILL website and affects any
visitor, including authenticated administrators.
Please check the attached video proof of concept, which demonstrates the complete exploitation process
and successful JavaScript execution.
Steps to Reproduce
. Install TYPEMILL 2.25.0 and with the provided bundle it comes with the theme Cyanine theme.
. Log in as an administrator.
. Create a new user and assign the Author role.
. Log in using the Author account.
. Create a new folder.
. Inside that folder, create a new page or post.
. Log back in as the administrator and publish both the parent folder and the page/post.
. From the administrator panel, open the Cyanine theme settings.
. Enable the setting that displays hero images in post listings:
Post Configuration (Blog or News) → Hero Images → Show hero images in all lists of posts
10. Save the Cyanine theme configuration.
11. Log in again using the Author account, open the folder, and navigate to the Meta tab. In the Folder
section, select Posts sorted by publish date for news or blogs , then save the changes.
12. Now open the page and go to meta tab
13. Upload a hero image (any allowed image)
14. Enter the following payload in the Hero Image Alt Text field:
. Save the page/post. No additional administrative review or publication is required.
. Visit the public URL of the parent folder.
. The payload executes, and a JavaScript alert displays the website’s domain.
Impact
An Author can execute arbitrary JavaScript in the browsers of visitors to the affected page. Depending on
the victim's session and privileges, this may enable actions performed in the victim's context, access to
sensitive page data, interface manipulation, phishing, or administrative account compromise.

---

# Implementation Plan Review

## Review Status

The current task file contains the vulnerability report only, but no actionable implementation plan exists yet. This review adds the missing plan and identifies related hardening opportunities before any code is changed.

## Root Cause Analysis

The `heroimagealt` value is stored in the page/folder YAML metadata (`/content/**/*.yaml`) without validation or sanitization. The bundled Cyanine theme then renders this value into the `alt` attribute of `<img>` tags using raw Twig interpolation:

- `themes/cyanine/partials/posts.twig` line 16: `alt="{{ post.meta.heroimagealt }}"`
- `themes/cyanine/home/landingpageNews.twig` lines 20 and 46: `alt="{{ page.meta.heroimagealt }}"` and `alt="{{ post.meta.heroimagealt }}"`

Twig auto-escapes variable output by default **unless** the value is embedded in an HTML attribute context that is already inside a raw string. In this case, because the attribute is built with a literal `alt="` prefix and no explicit escaping filter, a payload such as `"><img src=x onerror=alert(document.domain)>` breaks out of the `alt` attribute and injects new HTML/JS.

Additionally, the same templates omit escaping for other meta fields:

- `themes/cyanine/home/landingpageNews.twig` line 23/49: `{{ page.meta.title }}` and `{{ post.meta.title }}` (missing `|e`)
- `themes/cyanine/home/landingpageNews.twig` line 50: `{{ post.meta.author | default(post.meta.owner) }}` (missing `|e`)
- `themes/cyanine/landingpage.twig` lines 62/63/66: `{{ page.meta.shorttitle }}` and `{{ page.meta.description }}` (missing `|e`)

While these are not the primary reported vector, they are the same class of vulnerability and should be fixed in the same pass.

The backend (`ControllerWebFrontend.php`) already uses `htmlspecialchars($metaImageAlt, ENT_QUOTES, 'UTF-8')` for the `twitter:image:alt` meta tag, confirming that escaping is expected at the point of output.

## Proposed Implementation Plan

1. **Fix the reported vector in Cyanine theme templates**
   - Add Twig `|e('html_attr')` or `|e` filter to every `post.meta.heroimagealt` and `page.meta.heroimagealt` output.
   - The `html_attr` strategy is preferable for attribute contexts because it encodes all characters that are special in HTML attributes (including quotes).

2. **Harden related meta outputs in the same theme files**
   - Add `|e` to `post.meta.title`, `page.meta.title`, `post.meta.author`, `post.meta.owner`, `post.meta.description`, `page.meta.shorttitle` in the affected templates.
   - Keep existing escaped outputs intact (do not double-escape).

3. **Verify no other themes are affected**
   - Search `/themes` for `heroimagealt` usage. Already checked: only Cyanine uses it.
   - Search `/themes` for similar patterns of unescaped meta fields in HTML attributes and fix any found.

4. **Backend validation (defense in depth)**
   - Consider adding a strip/escape helper in `Meta.php` or the controller when reading metadata for public display. However, because the theme is the consumer of the data and the fix is minimal, the primary fix should remain in the templates.
   - Avoid removing the ability for legitimate alt text to contain special characters (e.g., single quotes).

5. **Testing / Verification**
   - Reproduce the reported payload: set `heroimagealt` to `"><img src=x onerror=alert(document.domain)>`.
   - Visit the parent folder page with "Show hero images in all lists of posts" enabled.
   - Confirm the payload is rendered as literal text inside the `alt` attribute and no JavaScript executes.
   - Inspect the HTML source to confirm the attribute is encoded as `alt="&quot;&gt;&lt;img src=x onerror=alert(document.domain)&gt;"`.
   - Run the existing frontend rendering tests (if any) to ensure no regressions.
   - Check landing page and blog page templates for the same payload behavior.

6. **Documentation**
   - Add a note in the Cyanine theme changelog or security notes about the fix.
   - Do not create new documentation files unless explicitly requested; only update existing task/changelog files if present.

## Files to Modify

- `themes/cyanine/partials/posts.twig`
- `themes/cyanine/home/landingpageNews.twig`
- `themes/cyanine/blog.twig` (escape alt attribute and switch to `heroimagealt` instead of `heroimage` for the alt text)
- Optionally review `themes/cyanine/landingpage.twig` for escaping consistency

## Files to Review but Likely Not Modify

- `system/typemill/Controllers/ControllerWebFrontend.php`: already escapes `heroimagealt` for meta tags; no change needed.
- `system/typemill/settings/metatabs.yaml`: field definition is correct; no change needed.

## Improvement Recommendations

- Consider adding a CI/static-analysis step to detect unescaped Twig variable output in HTML attributes, especially for meta fields that are editable by lower-privilege roles.
- Consider adding server-side validation to reject angle brackets, quotes, and other HTML-special characters in alt text fields at save time, as a defense-in-depth measure.

## Implementation Completed

The fix has been applied to the Cyanine theme files.

### Changes Made

- `themes/cyanine/partials/posts.twig`: added `|e('html_attr')` to `post.meta.heroimagealt`.
- `themes/cyanine/home/landingpageNews.twig`: added `|e('html_attr')` to `page.meta.heroimagealt` and `post.meta.heroimagealt`; added `|e` to `page.meta.title`, `post.meta.title`, `post.meta.author`, `post.meta.owner`, and the truncated description expressions.
- `themes/cyanine/blog.twig`: corrected the `alt` attribute to use `post.meta.heroimagealt` (was incorrectly using `post.meta.heroimage`) and added `|e('html_attr')` escaping.
- `themes/cyanine/landingpage.twig`: added `|e` to `title`, `page.meta.shorttitle`, and `page.meta.description`.

### Verification

- Parsed all modified Twig templates with the installed Twig engine; all templates compiled successfully.
- Rendered each modified template with mock data containing the reported payload (`"><img src=x onerror=alert(document.domain)>`) and other script-injected meta values; no raw payload or `<script>` tags appeared in the output.

### Status

The implementation has been completed and verified. The modified files are in the working tree and are ready for review or commit.
