Hello Typemill Security Team,

I am a security researcher and I have discovered a Stored Cross-Site
Scripting (XSS) vulnerability in Typemill CMS v2.x.

## Vulnerability Details

Type: Stored Cross-Site Scripting (XSS)
Severity: Medium-High (CVSS 7.6)
Auth Required: Administrator
Affected Version: v2.x (latest)

## Root Cause

The inlineLink() method in ParsedownExtension.php (line 934-981)
assigns user-supplied href values directly without validating or
filtering dangerous URI schemes such as javascript: or vbscript:.

Affected files:
- system/typemill/Extensions/ParsedownExtension.php (line 934-981)
- system/typemill/Extensions/TwigMarkdownExtension.php
- themes/cyanine/home/landingpageIntro.twig

## Steps to Reproduce

1. Login as admin
2. Go to: System → Themes → Cyanine → Configure
image.png
3. Expand "Landingpage Intro Segment"
4. In "Text for your landingpage intro (use markdown)" enter:
   [Click me](javascript:alert(document.domain))
image.png
5. Click Save
6. Visit homepage as any user
7. Click the "Click me" link
image.png
8. JavaScript executes → alert(document.domain) fires

## PoC Verification

Command used to verify:

docker exec typemill php -r '
  require "vendor/autoload.php";
  require "system/typemill/Extensions/ParsedownExtension.php";
  $p = new \Typemill\Extensions\ParsedownExtension("",[],null);
  echo $p->markup($p->text("[x](javascript:alert(1))"));
'

Output confirmed:
<p><a href="javascript:alert(1)">x</a></p>

## Impact

Any visitor who clicks the malicious link will execute
attacker-controlled JavaScript in their browser session, enabling:
- Admin session hijacking
- Credential theft
- Further attack chaining

## Suggested Fix

In ParsedownExtension.php inlineLink() method, add URI scheme
validation before assigning href:

$allowed_schemes = ['http', 'https', 'mailto', 'ftp'];
$scheme = parse_url($href, PHP_URL_SCHEME);
if ($scheme && !in_array(strtolower($scheme), $allowed_schemes)) {
    $href = '#';
}

---

## Implementation Plan & Security Review

### Current State

- A partial fix for the reported vector is already present in `system/typemill/Extensions/ParsedownExtension.php` at lines 981-986.
- The existing sanitizer only covers the direct inline-link branch (`[text](url)`).
- The original PoC `[x](javascript:alert(1))` is now neutralized to `<p><a href="#">x</a></p>`.

### Gap Analysis

Testing revealed that the reference-style link/image branch is **not** sanitized:

- `[x][ref]\n\n[ref]: javascript:alert(1)` still renders `<a href="javascript:alert(1)">x</a>`.
- `![x][ref]\n\n[ref]: javascript:alert(1)` still renders `<img src="javascript:alert(1)">`.

This means the current fix is incomplete and can be bypassed by any admin who knows the reference-style Markdown syntax.

### Proposed Fix

1. Extract the scheme whitelist logic into a small protected helper method, e.g. `sanitizeUrl($url)`, in `ParsedownExtension.php`.
2. Reuse that helper for **both** code paths in `inlineLink()`:
   - The direct inline link href after base/media URL rewriting.
   - The reference link href read from `$Definition['url']`.
3. Keep the existing allowlist `['http', 'https', 'mailto', 'ftp']` and preserve relative links and media links (no scheme = no change).
4. Do not introduce global state, new dependencies, or configuration changes.

### Files to Modify

- `system/typemill/Extensions/ParsedownExtension.php` (security bug fix; allowed per architecture rules).

No changes are required in `TwigMarkdownExtension.php` or `themes/cyanine/home/landingpageIntro.twig` because both already route Markdown rendering through `ParsedownExtension`.

### Verification Steps

Run these PHP CLI checks after the change:

```bash
php -r '
require "system/vendor/autoload.php";
require "system/typemill/Extensions/ParsedownExtension.php";
$p = new \Typemill\Extensions\ParsedownExtension("",[],null);

$cases = [
    "[x](javascript:alert(1))"                => "inline javascript",
    "[x](vbscript:alert(1))"                  => "inline vbscript",
    "[x](data:text/html,<script>alert(1)</script>)" => "inline data",
    "[x][ref]\n\n[ref]: javascript:alert(1)" => "reference javascript",
    "[x][ref]\n\n[ref]: vbscript:alert(1)"   => "reference vbscript",
    "![x][ref]\n\n[ref]: javascript:alert(1)" => "reference image",
    "[x](https://example.com)"                => "allowed https",
    "[x](mailto:test@example.com)"            => "allowed mailto",
    "[x](/relative/path)"                     => "relative link",
    "[x](media/file.jpg)"                       => "media link",
];

foreach ($cases as $md => $label) {
    echo str_pad($label, 35) . ": " . $p->markup($p->text($md)) . "\n";
}
'
```

Expected results:

- Malicious schemes must render as `href="#"` or `src="#"`.
- Allowed schemes (`http`, `https`, `mailto`, `ftp`) and relative/media links must remain unchanged.

### Additional Considerations

- `TwigMarkdownExtension.php` already instantiates `ParsedownExtension`, so it will inherit the fix automatically.
- `landingpageIntro.twig` uses the `markdown()` Twig function, which also routes through `ParsedownExtension`.
- The separate `introButtonLink` theme field at line 51 of `landingpageIntro.twig` is rendered raw (`href="{{ settings.themes.cyanine.introButtonLink }}"`). This is a different control and not part of the Markdown parser; it is a URL input field rather than the reported Markdown link vector. A separate hardening ticket may be warranted if Typemill wants to sanitize all theme URL fields as well.
- No database, JWT, new build steps, or core architectural changes are introduced.

### Confirmation Requested

Please confirm the proposed approach before code changes are applied. If you want the scope expanded to also sanitize the `introButtonLink` theme field, or to allow additional schemes such as `tel:`, let me know.

---

## Implementation Result

**Status:** Applied to `system/typemill/Extensions/ParsedownExtension.php`.

**Changes made:**

1. Added a protected `sanitizeUrl($url)` helper.
2. The helper uses Parsedown's existing `$safeLinksWhitelist` (which already includes user-defined schemes from the `urlschemes` setting) instead of a hardcoded allowlist.
3. Reused the helper for the direct inline link branch.
4. Reused the helper for the reference-style link/image branch.

**Why this avoids breaking the `urlschemes` setting:**

The constructor already adds user-defined schemes from `settings['urlschemes']` to `$this->safeLinksWhitelist`. By using that property, custom schemes like `dict://` are preserved while dangerous schemes like `javascript:` are still blocked.

**Verification:**

- Original PoC `[x](javascript:alert(1))` → `<p><a href="#">x</a></p>` ✅
- Inline `vbscript:`, `data:text/html`, `file:`, `jar:` → blocked ✅
- Reference link `[x][ref]` + `[ref]: javascript:alert(1)` → `<a href="#">x</a>` ✅
- Reference image `![x][ref]` + `[ref]: javascript:alert(1)` → `<img src="#">` ✅
- Allowed defaults: `https`, `http`, `mailto`, `ftp`, `tel`, `irc`, `data:image/png;base64,...` → allowed ✅
- Relative links (`/relative/path`) and media links (`media/file.jpg`) → preserved ✅
- Custom schemes from `urlschemes` setting (`dict://`, `myapp:`) → allowed ✅
- Mixed-case `Javascript:alert(1)` → blocked ✅
- `php -l` syntax check passed ✅
