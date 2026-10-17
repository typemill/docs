# Security Vulnerability Analysis & Implementation Plan

Date: 2026-05-03  
Analyst: AI Security Review  
Source: `/ai/tasks/security-fixes.md`

---

## Overview

Three vulnerability reports were reviewed. Each was verified against the actual source code.  
Findings range from confirmed and fixable to out-of-scope infrastructure issues.

---

## Report 1: `javascript:` URI in Markdown Links (Stored XSS)

### Claim

The `inlineLink()` method in `ParsedownExtension.php` (lines 934–981) assigns user-supplied
`href` values without validating the URI scheme, allowing `javascript:` payloads to be rendered
as clickable links.

### Code Verified

**File:** `system/typemill/Extensions/ParsedownExtension.php`, line 981

```php
# start typemill: if relative link or media-link
$href = $matches[1];
if($href[0] == '/')
{
    $href = $this->baseUrl . $href;
}
elseif(substr( $href, 0, 6 ) === "media/")
{
    $href = $this->baseUrl . '/' . $href;
}
# end typemill

$Element['attributes']['href'] = $href;  // ← no scheme validation here
```

The PoC `[x](javascript:alert(1))` produces `<a href="javascript:alert(1)">x</a>`.  
This was confirmed by reading the code — no scheme whitelist exists.

### Validation Gap

Theme settings store `introMarkdown` as a `textarea` field. The backend validation
(`recursiveValidation` → `field()` in `Validation.php`, line 926–936) applies the `noHTML`
rule to `textarea` types. The `noHTML` rule uses `strip_tags()`, which only catches HTML
tags (`<...>`). A markdown link like `[Click me](javascript:alert(1))` contains no HTML
tags and **passes `noHTML` validation undetected**.

### Verdict: CONFIRMED — Real vulnerability, limited attack surface

**Constraints on exploitability:**
- Writing to theme settings requires **manager role** (high privilege).
- This is effectively self-XSS for an administrator on their own instance in the default configuration.
- Risk increases significantly if any lower-privileged role is ever granted theme configuration
  write access, or if the `markdown()` Twig function is called with user-supplied content
  in a theme or plugin.

**Severity: Low–Medium (CVSS ~4.5 in default config; higher in multi-role deployments)**

---

## Report 2a: Unescaped `og:title` / `og:description` Output (Stored XSS)

### Claim

`Assets.php` renders meta tags without `htmlspecialchars()`, allowing an attacker to inject
HTML attributes by placing a double-quote (`"`) inside a meta title or description field.

### Code Verified

**File:** `system/typemill/Controllers/ControllerWebFrontend.php`, lines 347–349

```php
$assets->addMeta('og_site_name', '<meta property="og:site_name" content="' . $this->settings['title'] . '">');
$assets->addMeta('og_title',     '<meta property="og:title" content="'     . $metadata['meta']['title'] . '">');
$assets->addMeta('og_description','<meta property="og:description" content="'. $metadata['meta']['description'] . '">');
```

No `htmlspecialchars()` is applied. Twig's global autoescape is **disabled** in `system.php`:

```php
'autoescape' => false
```

These strings are written as raw HTML into the page `<head>`.

### Validation Gap

The meta `title` field is type `text` and `description` is type `textarea`. Both receive
the `noHTML` rule via `recursiveValidation`. However `noHTML` only strips HTML tags using
`strip_tags()`. It does **not** reject or encode double-quote characters (`"`).

A payload like:

```
XSS" onmouseover="alert(1)" x="
```

contains no HTML tags and **passes `noHTML` validation**. After storage it renders as:

```html
<meta property="og:title" content="XSS" onmouseover="alert(1)" x="">
```

### Practical Impact Assessment

The report's PoC triggers the injected handler via the browser DevTools console
(`document.querySelector(...).onmouseover()`). Meta tags are not interactive by default,
so **passive exploitation requires chaining** with a CSS breakout or another XSS vector.
This somewhat reduces the practical severity compared to a clickable link.

However, the vulnerability is real: the injected attribute is present in the DOM and can
be triggered programmatically. In a scenario where a user's browser has extensions or
scripts that iterate over meta tags (some SEO tools do), or in combination with other
vectors, this is meaningful.

**Who can exploit this:** Any authenticated user with content edit access (**author role
or above**, which is `mycontent/update` — contributor level). This is a much larger
attack surface than Report 1.

**Verdict: CONFIRMED — Real vulnerability, moderate attack surface**

**Severity: Medium (CVSS ~5.5–6.0)**

Additional `addMeta()` calls with dynamic content that should also be reviewed:

| Line | Field | Source |
|------|-------|--------|
| 347 | `og:site_name` | `$this->settings['title']` (system setting, manager-only write) |
| 350 | `og:type` | hardcoded `"article"` — safe |
| 351 | `og:url` | `$item->urlAbs` — derived from filesystem path, low risk |
| 380 | `og:image` | `$metaImageUrl` — image path from meta, should be encoded |
| 381 | `twitter:image:alt` | `$metaImageAlt` — user-supplied alt text, must be encoded |
| 486/492/498 | `alternate` hreflang | `$multilangData` values — should be encoded |

---

## Report 2b: No CSRF Protection on API Endpoints

### Claim

The API has no CSRF token requirement, allowing an attacker to trick an authenticated
administrator into making state-changing API requests from a malicious page.

### Code Verified

`slim/csrf` is installed and a `TwigCsrfExtension` exists for HTML form protection,
but **no CSRF middleware is applied to any `/api/v1` routes** in `routes/api.php`.

The API route group is protected only by `ApiAuthentication` middleware, which checks:
1. `X-Session-Auth` header + existing session attributes (for internal/Vue use), or
2. HTTP Basic Auth credentials (for external use).

### Assessment

The `X-Session-Auth: true` header acts as a **partial CSRF mitigation**. Cross-origin
HTML form submissions cannot set custom headers, and the `SameSite` cookie attribute
(if set) would also block simple CSRF. Classic `<form>`-based CSRF is therefore not
possible.

However, **`fetch()`-based CSRF from a page with an XSS vector remains possible**:
a script running in the victim's browser can set arbitrary headers and make
authenticated API requests. This creates a circular dependency: to exploit the CSRF
you first need an XSS, but fixing the XSS (Reports 1 and 2a) breaks the CSRF chain.

**Verdict: PARTIAL — Real but only exploitable in combination with XSS**

**Severity: Low (standalone); Medium (chained with XSS)**  
Fixing Reports 1 and 2a significantly reduces the practical risk of this issue.

---

## Report 3: DMARC Misconfiguration on typemill.net

### Claim

The `typemill.net` domain lacks a DMARC `quarantine` or `reject` policy, allowing
attackers to send spoofed emails using `@typemill.net` addresses.

### Assessment

**This is not a CMS code vulnerability.** It is a DNS/email infrastructure configuration
issue for the `typemill.net` domain and must be resolved at the DNS registrar or DNS
provider by adding a DMARC TXT record.

**Verdict: OUT OF SCOPE for this codebase. No code changes required.**

The fix (adding a `_dmarc.typemill.net` TXT record with `p=quarantine` or `p=reject`)
is a DNS operation, not a software change.

---

## Implementation Plan

### Fix 1 — URI Scheme Whitelist in `ParsedownExtension::inlineLink()`

**File:** `system/typemill/Extensions/ParsedownExtension.php`  
**Location:** After `$href` is computed (around line 980), before it is assigned to
`$Element['attributes']['href']`  
**Priority:** Medium  
**Risk:** Low — purely additive check, no behavior change for valid URLs

**Change:**

```php
# start typemill: if relative link or media-link
$href = $matches[1];
if($href[0] == '/')
{
    $href = $this->baseUrl . $href;
}
elseif(substr( $href, 0, 6 ) === "media/")
{
    $href = $this->baseUrl . '/' . $href;
}
# end typemill

// ADD: block dangerous URI schemes
$scheme = parse_url($href, PHP_URL_SCHEME);
if ($scheme !== null && !in_array(strtolower($scheme), ['http', 'https', 'mailto', 'ftp'], true))
{
    $href = '#';
}

$Element['attributes']['href'] = $href;
```

**Notes:**
- `parse_url()` returns `null` for relative URLs (no scheme) — those are safe and pass through.
- Empty string scheme also passes through (relative `#anchor` links etc.).
- `data:` and `vbscript:` are also blocked by this whitelist.

---

### Fix 2 — Output Encoding for Meta Tags in `ControllerWebFrontend`

**File:** `system/typemill/Controllers/ControllerWebFrontend.php`  
**Location:** Lines 347–351 and related `addMeta()` calls  
**Priority:** High  
**Risk:** Very low — `htmlspecialchars(ENT_QUOTES)` is the correct and standard fix for
HTML attribute context; it does not change the visible content for legitimate values

**Changes:**

Replace the unescaped og: meta tag block (around line 347):

```php
// BEFORE
$assets->addMeta('og_site_name', '<meta property="og:site_name" content="' . $this->settings['title'] . '">');
$assets->addMeta('og_title',     '<meta property="og:title" content="'     . $metadata['meta']['title'] . '">');
$assets->addMeta('og_description','<meta property="og:description" content="'. $metadata['meta']['description'] . '">');
$assets->addMeta('og_type',      '<meta property="og:type" content="article">');
$assets->addMeta('og_url',       '<meta property="og:url" content="' . $item->urlAbs . '">');

// AFTER
$assets->addMeta('og_site_name', '<meta property="og:site_name" content="' . htmlspecialchars($this->settings['title'], ENT_QUOTES, 'UTF-8') . '">');
$assets->addMeta('og_title',     '<meta property="og:title" content="'     . htmlspecialchars($metadata['meta']['title'], ENT_QUOTES, 'UTF-8') . '">');
$assets->addMeta('og_description','<meta property="og:description" content="'. htmlspecialchars($metadata['meta']['description'], ENT_QUOTES, 'UTF-8') . '">');
$assets->addMeta('og_type',      '<meta property="og:type" content="article">');
$assets->addMeta('og_url',       '<meta property="og:url" content="' . htmlspecialchars($item->urlAbs, ENT_QUOTES, 'UTF-8') . '">');
```

Also fix the image alt and hreflang calls (around lines 380–381 and 486–498):

```php
// BEFORE
$assets->addMeta('twitter_image_alt', '<meta name="twitter:image:alt" content="' . $metaImageAlt . '">');

// AFTER
$assets->addMeta('twitter_image_alt', '<meta name="twitter:image:alt" content="' . htmlspecialchars($metaImageAlt, ENT_QUOTES, 'UTF-8') . '">');
```

```php
// BEFORE (hreflang alternate links, lines ~486–498)
$assets->addMeta('alternate_' . $project['id'], '<link rel="alternate" hreflang="' . $project['id'] . '" href="' . $urlinfo['baseurl'] . $multilangData[$project['id']] . '" />');

// AFTER
$assets->addMeta('alternate_' . $project['id'], '<link rel="alternate" hreflang="' . htmlspecialchars($project['id'], ENT_QUOTES, 'UTF-8') . '" href="' . htmlspecialchars($urlinfo['baseurl'] . $multilangData[$project['id']], ENT_QUOTES, 'UTF-8') . '" />');
```

---

### Fix 3 — CSRF (Deferred / Optional)

**Priority:** Low (only exploitable in combination with an existing XSS)  
**Recommendation:** Defer. Fixing Reports 1 and 2a removes the practical CSRF chain.

If CSRF protection is desired in the future, the recommended approach is:
- Add a per-session CSRF token to the session on login.
- Require the token as a header (e.g. `X-CSRF-Token`) on all state-changing API requests
  (POST, PUT, DELETE) made via session authentication.
- The Vue.js layouts already inject `X-Session-Auth`; adding a second header is trivial.
- Basic Auth requests are inherently CSRF-safe and would be excluded from this check.

This is an **architectural change** and should be planned as a separate task.

---

### Fix 4 — DMARC (Out of Scope)

Add a DNS TXT record at `_dmarc.typemill.net`:

```
_dmarc.typemill.net. IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@typemill.net"
```

This is a **DNS operation**, not a code change. Must be done by the domain owner.

---

## Summary

| # | Vulnerability | Confirmed | Severity | Fix Required | Fix Location |
|---|---------------|-----------|----------|--------------|-------------|
| 1 | `javascript:` URI in Markdown links | Yes | Low–Medium | Yes | `ParsedownExtension.php` |
| 2a | Unescaped `og:title` / `og:description` | Yes | Medium | Yes | `ControllerWebFrontend.php` |
| 2b | No CSRF on API endpoints | Partial | Low | Deferred | Architecture |
| 3 | DMARC misconfiguration | Out of scope | — | No (DNS only) | DNS / domain registrar |

**Recommended implementation order:** Fix 2a first (widest attack surface, simplest fix),
then Fix 1 (completeness), then evaluate Fix 3 as a future hardening task.
