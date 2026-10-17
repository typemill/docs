# PlantUML Plugin Code Review

## Overview

The plugin works functionally and the event integration (`onHtmlLoaded`, `onCspLoaded`, `onTwigLoaded`) is well thought out. However, there are several issues ranging from architectural violations to reliability bugs and minor code-quality concerns.

---

## 1. Critical: Cache location violates Typemill architecture

**File:** `plantuml.php` (line 287, 313)

**Problem:** Downloaded images are stored in `/plugins/plantuml/temp/` and served from there.

**Why this matters:** Per the Typemill architecture (`/ai/docs/architecture.md`):
- *"Plugins store related data in the `/data` folder"*
- *"Cache and `/data` contents must always be disposable."*
- The `/cache` directory is explicitly designated for *"Temporary public cache files"*.

PlantUML rendered images are derived, disposable, public cache files. Storing them inside the plugin directory means:
- Cache is not disposable (deleting `/cache` or `/data` won't clean it up).
- The plugin ships a `.gitignore` for `temp/` instead of using core conventions.
- The URL path is hardcoded to `/plugins/plantuml/temp/`.

**Fix:** Use `/cache/plantuml/` (or `/data/plantuml/` if persistence is desired, but `/cache/` is more appropriate). Use `$baseurl . '/cache/plantuml/' . $filename` for the URL. The core `Storage` model supports `cacheFolder` as a writable target.

---

## 2. High: Redundant `require_once` for autoloaded class

**File:** `plantuml.php` (line 265)

**Problem:**
```php
require_once __DIR__ . '/PlantUmlEncoder.php';
$encoder = new PlantUmlEncoder();
```

**Why this matters:** `composer.json` autoloads `"Plugins\\": "plugins/"` via PSR-4. `PlantUmlEncoder` is in namespace `plugins\plantuml` and is already autoloaded. The `require_once` causes an unnecessary filesystem `stat` on every diagram render.

**Fix:** Remove the `require_once` line entirely.

---

## 3. High: No error handling when the PlantUML server is unreachable

**File:** `plantuml.php` (lines 298–309)

**Problem:**
```php
$imageContent = file_get_contents($imageUrl, false, $context);
if ($imageContent !== false) {
    file_put_contents($localPath, $imageContent);
}
// ...
return '<figure ...><img src="' . $localUrl . '" ... /></figure>';
```

If the remote server is down, returns an error, or the network is unavailable, `$imageContent` is `false`. The plugin then returns HTML referencing a file that was never written. PDF/eBook generators will embed a broken image.

**Fix:** If `file_get_contents` fails, return a fallback (e.g., the original `<pre>` code block or an error placeholder) so the user sees something rather than a broken image reference.

**Note Sebastian:** I usually check first if curl is available since file_get_contents is blocked by many hosting providers.

---

## 4. Medium: `allow_url_fopen` not checked

**File:** `plantuml.php` (line 305)

**Problem:** Remote fetching relies on `file_get_contents($imageUrl, ...)`. If `allow_url_fopen` is disabled (common in hardened PHP setups), the fetch silently fails.

**Fix:** Check `ini_get('allow_url_fopen')` before attempting the fetch. If disabled, return a clear fallback or error message.

**Note Sebastian:** I guess that allow_url_fopen is also often blocked by hosting providers.

---

## 5. Medium: CSP whitelisting ignores non-standard ports

**File:** `plantuml.php` (lines 45–51)

**Problem:**
```php
$parsedUrl = parse_url($serverUrl);
if (isset($parsedUrl['host'])) {
    $data[] = $parsedUrl['host'];
}
```

If a user runs a private PlantUML server at `http://localhost:8080`, CSP whitelists `localhost` (implicit port 80), but the browser actually requests `localhost:8080`. The images will be blocked by the browser's CSP.

**Fix:** Append the port if it exists and is non-standard:
```php
$host = $parsedUrl['host'];
if (isset($parsedUrl['port'])) {
    $host .= ':' . $parsedUrl['port'];
}
$data[] = $host;
```

---

## 6. Medium: Cache files accumulate forever (cache bloat)

**File:** `plantuml.php` (lines 294, 298)

**Problem:** The cache filename is `md5($imageUrl)`. If the user changes the server URL, output format, border color, or transparency setting, the MD5 changes and a *new* file is generated. Old files are never cleaned up.

**Fix:** Implement a simple garbage collection mechanism (e.g., delete cache files older than X days on a cache miss). Alternatively, document the need for manual pruning.

---

## 7. Low: Unused method `generatePlantUmlImage`

**File:** `plantuml.php` (lines 273–277)

**Problem:** The method is defined but never called anywhere in the codebase.

**Fix:** Remove the dead code.

---

## 8. Low: Redundant regex group `(?:.*?)?`

**File:** `plantuml.php` (lines 68, 137)

**Problem:** The regex uses `(?:.*?)?`, which is a non-capturing optional group wrapping an already optional lazy match. It is semantically equivalent to `.*?` and adds noise.

**Fix:** Simplify to `/<pre><code class="language-plantuml-diagram(.*?)"[^>]*>\s*(.*?)\s*<\/code><\/pre>/ms`.

---

## 9. Low: Hardcoded plugin path in generated URLs

**File:** `plantuml.php` (line 313)

**Problem:**
```php
$localUrl = $baseUrl . '/plugins/plantuml/temp/' . $fileName;
```

If the plugin folder is ever renamed, all cached image URLs break.

**Fix:** Derive the path dynamically, or (better) move the cache to `/cache/plantuml/` as recommended in issue #1.

---

## 10. Low: No network timeout on remote fetch

**File:** `plantuml.php` (line 299)

**Problem:** `file_get_contents` uses PHP's default socket timeout (usually 60 seconds). If the PlantUML server is slow or unresponsive, page rendering blocks for up to a minute.

**Fix:** Add a `timeout` to the stream context:
```php
$context = stream_context_create([
    "http" => [
        "header" => "User-Agent: Typemill-PlantUML-Plugin\r\n",
        "timeout" => 10
    ]
]);
```

## Summary of Recommended Fixes

| Priority | Issue | Action | Status |
|---|---|---|---|
| Critical | Cache in `/plugins/plantuml/temp/` | Move to `/cache/generated/plantuml/` via `generateStaticAsset()` | **Fixed** |
| High | Redundant `require_once` | Removed | **Fixed** |
| High | No fetch error handling | `fetchRemoteImage()` returns `false`; `renderStaticPlantUml()` returns fallback SVG | **Fixed** |
| Medium | `allow_url_fopen` unchecked | `fetchRemoteImage()` checks `ini_get('allow_url_fopen')` and falls back to curl | **Fixed** |
| Medium | CSP port bug | Appends non-standard port in `onCspLoaded()` | **Fixed** |
| Medium | Cache bloat | `generateStaticAsset()` uses SHA1 hash; old files still accumulate but are in `/cache/generated/` which is disposable | **Partially fixed** |
| Low | Unused method | `generatePlantUmlImage()` removed | **Fixed** |
| Low | Regex noise | Simplified `(?:.*?)?` to `[^>]*` | **Fixed** |
| Low | Hardcoded path | `generateStaticAsset()` builds URL dynamically from container `baseurl` | **Fixed** |
| Low | No timeout | `fetchRemoteImage()` applies 10s timeout to both `file_get_contents` and curl | **Fixed** |

---

## Additional Notes

- The `PlantUmlEncoder` implementation is correct and mirrors the official PlantUML encoding spec. No issues found there.
- The plugin correctly uses `onHtmlLoaded` (rather than `onMarkdownLoaded`) to avoid altering raw content in the Typemill Author editor. This is the right choice.
- The separation of `processHtmlForBrowser` (frontend JS) and `generateLocalPlantUmlHtml` (eBook/PDF static image) is architecturally sound.

---

## Evaluation: Using `generateStaticAsset()` in the PlantUML Plugin

*(Added after Typemill core introduced `Plugin::generateStaticAsset()` and `Plugin::getStaticAssetPath()` for the export-safe HTML pipeline.)*

### What the new helper offers

`generateStaticAsset()` handles:
- Deterministic SHA1 hashing
- Cache directory creation under `/cache/generated/{namespace}/`
- Skipping the generator when the file already exists
- Returning a public URL with the correct `baseurl`

This directly fixes:
- **#1 Critical — Cache location** → `/cache/generated/plantuml/` is the correct path.
- **#9 Low — Hardcoded path** → The helper constructs the URL dynamically from the container's `baseurl`.

### How the PlantUML plugin would use it

The plugin needs an `onExportHtmlLoaded` handler (currently missing). Because `onHtmlLoaded` already transforms `<pre><code>` into `<div class="plantuml-browser-render">`, the export handler should target those divs:

```php
public static function getSubscribedEvents()
{
    return [
        'onHtmlLoaded'       => 'onHtmlLoaded',
        'onExportHtmlLoaded' => 'onExportHtmlLoaded',
        'onCspLoaded'        => 'onCspLoaded',
        'onTwigLoaded'       => 'onTwigLoaded',
    ];
}

public function onExportHtmlLoaded($event)
{
    $data = $event->getData();
    $html = is_array($data) ? $data['html'] : $data;

    // Match the browser-render divs injected by onHtmlLoaded
    $regex = '/<div class="plantuml-browser-render"\s+data-plantuml-url="(.*?)"\s*(.*?)>(.*?)<\/div>/ms';

    $html = preg_replace_callback($regex, function ($matches) {
        $imageUrl = htmlspecialchars_decode($matches[1], ENT_QUOTES);
        $attrs    = $this->parseAttributes($matches[2]);

        // Decode the PlantUML URL back to raw code
        $code = $this->decodePlantUmlUrl($imageUrl);

        // Build a composite cache key that includes all rendering parameters
        $settings = $this->getPluginSettings('plantuml');
        $cacheKey = json_encode([
            'code'        => $code,
            'server_url'  => $settings['server_url']  ?? 'https://www.plantuml.com/plantuml',
            'format'      => $settings['output_format'] ?? 'svg',
            'transparent' => $settings['transparent_background'] ?? false,
            'border'      => $settings['border_color'] ?? '',
        ]);

        $extension = $settings['output_format'] ?? 'svg';

        $url = $this->generateStaticAsset(
            $cacheKey,
            function ($key) {
                $params = json_decode($key, true);
                $remoteUrl = $this->generatePlantUmlUrl($params['code'], $params);

                $context = stream_context_create([
                    'http' => [
                        'header'  => "User-Agent: Typemill-PlantUML-Plugin\r\n",
                        'timeout' => 10,
                    ]
                ]);

                $content = @file_get_contents($remoteUrl, false, $context);

                if ($content === false) {
                    // Fallback: return a transparent 1x1 pixel or error SVG
                    return '<svg xmlns="http://www.w3.org/2000/svg" width="100" height="20"><text x="0" y="15">PlantUML render error</text></svg>';
                }

                return $content;
            },
            $extension
        );

        return $this->buildFigureHtml($url, $attrs);
    }, $html);

    is_array($data)
        ? $event->setData(['html' => $html] + $data)
        : $event->setData($html);
}
```

### What is NOT solved by `generateStaticAsset()` alone

`generateStaticAsset()` handles caching, hashing, and path generation, but the plugin developer must still implement:

- **Network error handling** inside the generator callable (fallback SVG on fetch failure)
- **`allow_url_fopen` / curl fallback** inside the generator callable
- **Timeout configuration** inside the generator callable
- **Composite cache key** that includes all rendering parameters (not just the code)

These are all now implemented in the refactored plugin.

### Special concern for PlantUML: cache-key correctness

Unlike Mermaid or Math, a PlantUML diagram depends on **external server settings** (server URL, format, border color, transparency). If the user changes any of these, the cached image must be invalidated.

`generateStaticAsset()` hashes only the `$source` string. The plugin therefore encodes **all rendering parameters** into the source via `json_encode`, not just the raw PlantUML code. This ensures that changing the server URL or format produces a new hash and a fresh cached asset.

### Implementation Result

All items have been implemented in `plugins/plantuml/plantuml.php`:

- ✅ Added `onExportHtmlLoaded` subscription and handler.
- ✅ Removed the commented-out Twig filter, `processHtmlContent()`, `processRawMatchesLocalHtml()`, `processHtmlMatches()`, `generateLocalPlantUmlHtml()`, and `generatePlantUmlImage()`.
- ✅ Removed redundant `require_once __DIR__ . '/PlantUmlEncoder.php';`.
- ✅ `renderStaticPlantUml()` builds a composite JSON cache key with code + all rendering parameters and delegates to `generateStaticAsset()`.
- ✅ `fetchRemoteImage()` handles `allow_url_fopen`, curl fallback, 10-second timeout, and HTTP status checks.
- ✅ `onCspLoaded()` now appends non-standard ports.
- ✅ Regex in `onHtmlLoaded()` simplified from `(?:.*?)?` to `[^>]*`.
- ✅ `generatePlantUmlUrl()` refactored to accept an optional `$params` array for use by the generator callable.

### Verification

13 automated checks were run against the refactored plugin:

1. `onExportHtmlLoaded` is subscribed
2. Dead code (`processHtmlContent`, `generatePlantUmlImage`, `generateLocalPlantUmlHtml`) is removed
3. `renderStaticPlantUml()` exists with composite cache key
4. `fetchRemoteImage()` exists with error handling
5. CSP port bug fixed (verified with `localhost:8080`)
6. Regex simplified (`(?:.*?)?` removed, `[^>]*` present)
7. Uses `generateStaticAsset()`, no hardcoded `/plugins/plantuml/temp/` path
8. `require_once` removed
9. Browser rendering (`onHtmlLoaded`) produces correct divs with original-code backup
10. Export rendering transforms browser divs into static `<img>` tags in `/cache/generated/plantuml/`
11. Raw export (skipping `onHtmlLoaded`) also works on raw `<pre><code>` blocks
12. `buildFigureHtml()` correctly applies align, padding, and size attributes
13. `fetchRemoteImage()` returns `false` on unreachable URLs, and the plugin renders a fallback SVG

### Verdict

All 5 fixes from the original Verdict are now implemented:

1. ✅ **Add `onExportHtmlLoaded` subscription** — Added to `getSubscribedEvents()` and fully implemented.
2. ✅ **Remove commented-out Twig filter and `processHtmlContent()`** — Removed along with `processRawMatchesLocalHtml()`, `processHtmlMatches()`, `generateLocalPlantUmlHtml()`, and `generatePlantUmlImage()`.
3. ✅ **Build composite cache key** — `renderStaticPlantUml()` encodes code + server URL + format + transparency + border color into a JSON cache key.
4. ✅ **Handle network errors, `allow_url_fopen`, and timeouts** — `fetchRemoteImage()` checks `allow_url_fopen`, falls back to curl, applies 10s timeout, and validates HTTP status. The generator callable returns a fallback SVG on failure.
5. ✅ **Fix CSP port bug, regex noise, and dead code** — Port appended in `onCspLoaded()`, regex simplified to `[^>]*`, all dead methods removed.

**All 10 issues from the original code review are now addressed.**

The only remaining concern is **#6 Cache bloat**: old files still accumulate when settings change, but they now live in `/cache/generated/plantuml/` which is architecturally disposable. A future core enhancement (e.g. `clearStaticAssets()`) or periodic manual cleanup can address this.
