I am privately reporting a stored cross-site scripting vulnerability
in TypeMill's SVG sanitizer. The issue was reproduced against TypeMill
v2.26.0 in an isolated local environment.

An authenticated user with the contributor role can upload and publish
an SVG containing an xml-stylesheet processing instruction.
SvgSanitizer.php checks XML elements and attributes but does not
remove processing instructions. The instruction survives sanitization,
and the resulting SVG is served as image/svg+xml from the TypeMill
origin.

When an administrator opens the direct Live, Original, or Thumb SVG
link, the browser performs the XSLT transformation and executes
JavaScript in TypeMill's origin. In the local proof of concept, the
stored SVG used the administrator's existing session to call POST
/api/v1/user and create a harmless marker account. No data was
exfiltrated and no destructive action was performed.

Suggested classification:
- CWE-79
- High severity
- CVSS 3.1: CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:H/I:H/A:N (8.7)

Confirmed affected version: v2.26.0
First affected official release identified from Git history: 2.6.0
Configuration prerequisite: the official "Allow svg" setting is enabled

The complete private report is attached as:
TYPEMILL-2.26.0-SVG-XSLT-XSS-PRIVATE-REPORT.md
SHA-256: db4a20da909e3792f984f3a7ff4f92f482438ee4ee4114cd4ca0c0344b242040

The reproducible PoC package is attached as:
TYPEMILL-2.26.0-SVG-XSLT-POC.zip
SHA-256: 09b69ede5ad53f904f867fbbdb03df262f0c406616ad9a1bba33f3ce3406fbb7

Please confirm receipt. I am available to retest a proposed patch and
provide additional private evidence if needed. Please credit Fernando
Viana, GitHub username capivara-research, as finder/reporter in any
resulting advisory or CVE record.

Regards,
Fernando Viana
GitHub: https://github.com/capivara-research

---

## Implementation Plan & Security Review

### Report Triage

- The referenced attachments (`TYPEMILL-2.26.0-SVG-XSLT-XSS-PRIVATE-REPORT.md`, `TYPEMILL-2.26.0-SVG-XSLT-POC.zip`) are **not present** in the repository; only this task description exists. The vulnerability was therefore re-verified independently against the current codebase (see PoC below).
- The report is **confirmed and reproducible**. Classification as CWE-79 / High (8.7) is plausible; PR:L (contributor upload), UI:R (admin must open a direct media link), S:C (script runs in Typemill origin) all check out.
- The "first affected 2.6.0" claim is plausible (sanitizer predates current git history) but was not fully verified; it does not change the fix.

### Root Cause Analysis

`SvgSanitizer::sanitize()` (system/typemill/Models/SvgSanitizer.php:71-105) only iterates element nodes via `getElementsByTagName("*")` (DOM node type 1). Non-element node types are never removed and are re-serialized verbatim by `saveXML()`:

- Node type 7 (ProcessingInstruction): `<?xml-stylesheet type="text/xsl" href="data:..."?>` survives → browsers perform the XSLT transformation and execute JavaScript in the Typemill origin when the SVG is opened directly. **This is the reported vector.**
- Node type 10 (DocumentType): DOCTYPE and internal entity declarations survive (XXE / entity-expansion risk surface for any downstream consumer or older libxml on user hosts; PHP 8 + libxml 2.9+ does not expand external entities at parse time by default, so this is defense-in-depth, not an active bug in the parse step).
- Node type 8 (Comment): survives; harmless in modern browsers, but should be stripped for hygiene.

Serving context: `/media/live`, `/media/original`, `/media/thumb`, `/media/custom` are delivered **directly by the web server** (no Slim middleware runs, so `CspHeadersMiddleware` never applies to them). Only `/media/files/*` is routed through Slim (`ControllerWebDownload::download`, web.php:67) and gets `Content-Disposition: attachment`. Consequently the fix must happen at sanitization time (upload is the single point of truth); header-based mitigations would be Apache-only and cannot be the primary fix.

### Additional Defects Found During Review

1. **Dead error handling in `loadSVG()`** (SvgSanitizer.php:66-68): the method assigns `$result` but returns nothing (`null`). The `$loaded === false` checks in Media.php (lines 212-217 and 347-352) are unreachable dead code. Malformed SVG triggers PHP warnings and the pipeline continues; `saveXML()` on an empty DOMDocument returns just the XML declaration (not `false`), so corrupt files can be stored as near-empty SVGs.
2. **`loadXML()` called without flags**: no `LIBXML_NONET` (external DTD fetch attempts during parse) and no `libxml_use_internal_errors(true)` (libxml warnings can leak into logs/output). Hardening, not an active bug.
3. **Duplicated sanitizer block** in `Media.php::storeFile()` and `Media.php::checkAllowedExtension()` — identical 18-line blocks; a fix applied to one can be missed in the other. Extract into one private helper.
4. Whitelist spot-check (no change needed): no `xlink:href`/`href` is whitelisted on any element (blocks `a`/`use`/`image` URL references), event-handler attributes are not whitelisted, `<script>`/`<handler>`/`<listener>` are not whitelisted, and namespaced elements (e.g. `se:script`) fail the exact-match key check and get removed. The `<style>` element allows arbitrary CSS (no JS execution in modern browsers when served as image) — accepted limitation, out of scope.

### PoC Verification

Reproduced with the current codebase (in-memory, nothing stored):

```bash
php /tmp/opencode/svgtest/poc.php   # requires SvgSanitizer.php only
```

Output (PHP 8.5.10):

```
loadSVG return value: NULL          <-- confirms dead error handling
PI survives sanitization: YES - VULNERABLE
DOCTYPE/ENTITY survives: YES
```

The sanitized output still contains the complete `<?xml-stylesheet ...?>` processing instruction verbatim, including the XSLT `data:` URI with `<script>alert(document.domain)</script>`.

### Proposed Fix

All changes are security bug fixes in core, which is explicitly allowed ("Do not modify /system/typemill for new features (only for bug fixes)"). No architecture, no plugins, no new dependencies.

**1. `system/typemill/Models/SvgSanitizer.php`**

- `loadSVG()`: enable `libxml_use_internal_errors(true)`, call `loadXML($string, LIBXML_NONET)`, actually `return` the bool result, then `libxml_clear_errors()` and restore internal-errors state. Never use `LIBXML_NOENT` (it would expand entities).
- `sanitize()`: add a node-hygiene pass before the element whitelist loop:
  - Remove every `DOMProcessingInstruction` and `DOMComment` node, both at document level and nested inside the SVG root (recursive `childNodes` walk or XPath `//processing-instruction()|//comment()`; must also cover document-level siblings of the root element).
  - Reject (return failure) any document whose `doctype` is non-null, OR strip the DOCTYPE node. **Decision needed:** I recommend *reject with error message* — legitimate exported SVGs virtually never need a DOCTYPE, and rejecting is safer than silently rewriting. Alternative: strip it silently.
- Element/attribute whitelist logic stays unchanged (no regression risk for legitimate SVGs).

**2. `system/typemill/Models/Media.php`**

- Extract the duplicated SVG sanitization block from `storeFile()` (lines 208-228) and `checkAllowedExtension()` (lines 343-363) into one private helper (e.g. `sanitizeSvgFiledata()`), used by both call sites. Behavior identical, but the fixed logic exists exactly once. With `loadSVG()` returning bool, the existing error branches finally become reachable and are kept.

**3. Optional hardening — needs explicit confirmation**

- `.htaccess`: add a `<FilesMatch "\.(?i:svg)$">` block for the media folders setting `Content-Security-Policy: script-src 'none'; style-src 'unsafe-inline'; img-src 'self'` (blocks script execution even if some future sanitizer bypass allows XSLT through). Apache-only; nginx users would need manual config (document in release notes). This is defense-in-depth and NOT a substitute for the sanitizer fix.
- Not proposed: `Content-Disposition: attachment` for SVGs (breaks legitimate direct viewing of SVG media).

### Files to Modify

- `system/typemill/Models/SvgSanitizer.php` (required)
- `system/typemill/Models/Media.php` (required, dedupe only)
- `.htaccess` (optional, only on confirmation)

### Verification Plan (after implementation, before commit)

No PHPUnit in the repo; CLI verification matching the convention of `done-v2-26-security-xss-inline-link.md`:

1. `php -l` on both modified files.
2. CLI test matrix against `SvgSanitizer` directly:
   - Reported PoC (xml-stylesheet PI with XSLT `data:` URI) → PI must be gone.
   - PI and comment nested inside the `<svg>` root → removed.
   - DOCTYPE with internal entity → rejected with error (or stripped, per decision).
   - Malformed XML (`<svg><circle`) → `loadSVG()` returns `false`, error branch in Media.php reachable.
   - Valid SVG (circle/path/gradient/styles) → output functionally unchanged (elements and whitelisted attributes preserved).
   - `<script>` element, `onload` attribute, `xlink:href` attribute, namespaced `se:script` → still removed (no whitelist regression).
   - Round-trip check that a sanitized legit SVG still opens in a browser.
3. Optional manual regression: upload a legitimate SVG via the author UI with `allowsvg` enabled.

### Additional Considerations

- Both attack entry points are covered by one model fix: image upload (`ControllerApiImage::saveImage` → `Media::checkAllowedExtension`, gated by the `allowsvg` setting) and file upload (`ControllerApiFile` → `Media::storeFile`, `image/svg+xml` is in the allowed mtypes map independently of `allowsvg`).
- Already-stored malicious SVGs in existing installations will NOT be cleaned retroactively by the code fix. Worth mentioning in release notes (site owners can re-upload or delete suspicious media); no mass migration script proposed for a flat-file CMS.
- Public disclosure coordination, CVE record, and finder credit (Fernando Viana / capivara-research) are process items outside this code plan; recommend crediting as requested in the advisory.

### Confirmation Requested

Please confirm before any code is applied:

1. Proceed with the sanitizer + Media.php fix as described?
2. DOCTYPE handling: **reject upload with error** (recommended) or strip silently?
3. Include the optional `.htaccess` CSP hardening for SVG files (Apache-only)?

---

## Implementation Result

**Status:** Applied. Confirmed decisions: (1) proceed with fix, (2) DOCTYPE → reject upload with a meaningful error message, (3) no Apache-only CSP hardening (`.htaccess` unchanged).

**Changes made:**

1. `system/typemill/Models/SvgSanitizer.php`
   - `loadSVG()`: now enables `libxml_use_internal_errors(true)` (no libxml warnings leak), parses with `LIBXML_NONET` (no network access during parse), and **actually returns** the bool result of `loadXML()` (previously dead code). Never uses `LIBXML_NOENT`.
   - `sanitize()`: rejects the document (returns `false`) when a DOCTYPE/DTD declaration is present; removes all processing instructions (`XML_PI_NODE`, 7) and comments (`XML_COMMENT_NODE`, 8) at document level and nested anywhere via a new private `removeProcessingInstructionsAndComments()` stack-based walk; returns `true` on success. Element/attribute whitelist logic untouched.
2. `system/typemill/Models/Media.php`
   - The duplicated 18-line SVG sanitizer blocks in `storeFile()` and `checkAllowedExtension()` were extracted into one private `sanitizeSvgFiledata()` helper used by both entry points.
   - New error message for DOCTYPE rejection: `SVG files with a DOCTYPE (DTD) declaration are not allowed for security reasons. Please remove the DOCTYPE declaration and upload the plain SVG again.`
   - The previously unreachable error branches (load failure / save failure) are now reachable and unchanged in behavior.

**Verification (CLI, PHP 8.5.10, all 35 checks passed):**

- Reported PoC (`<?xml-stylesheet?>` PI with XSLT `data:` URI): loadSVG true, sanitize true, PI and payload completely removed from output, `<circle>` preserved. ✅
- Nested PI inside `<svg>` root and trailing document-level PI: removed. ✅
- Comments (document level and nested): removed. ✅
- DOCTYPE + internal entity: `sanitize()` returns `false`, Media rejects with the meaningful DOCTYPE message. ✅
- Malformed XML and undefined entity references: `loadSVG()` returns `false`, Media rejects with "could not load" message, no PHP warnings leaked. ✅
- Valid SVG (gradient, style, path, text, transform, viewBox, class, font-size): output functionally unchanged, re-parses as well-formed XML. ✅
- Whitelist regression suite: `<script>`, namespaced `se:script`, `onload`, `xlink:href` still removed; whitelisted elements/attributes preserved. ✅
- `php -l` clean on both files. ✅

**Notes:**

- One correction during implementation: PHP's constant for processing-instruction nodes is `XML_PI_NODE` (7); comment nodes are `XML_COMMENT_NODE` (8). `XML_PROCESSING_INSTRUCTION_NODE` does not exist and was caught by the test run.
- Already-stored SVGs in existing installations are not retroactively cleaned; re-upload suspicious media after updating. Worth a release-note line.
- Both upload entry points (image upload with `allowsvg` enabled, and file upload via `image/svg+xml` mtype) are covered by the single model fix.
- No changes to `.htaccess`, no new dependencies, no architecture changes (core bug fix only, allowed per constraints).