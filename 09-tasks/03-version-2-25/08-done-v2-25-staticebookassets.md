# Implementation Plan: Export-Safe HTML Rendering for Typemill

## Goal

Introduce a dedicated plugin event for generating export-safe HTML that can be used by:

- EPUB generation
- PDF generation (WeasyPrint, DomPDF, etc.)
- Static site exports
- Future export formats

The solution should avoid Twig filters entirely and allow plugins such as:

- PlantUML
- Mermaid
- MathJax / KaTeX
- Chart.js
- Graphviz
- D2

to replace dynamic browser content with static export-friendly content.

---

# Architecture

## Existing Web Rendering Pipeline

```text
Markdown
  ↓
Parsedown
  ↓
HTML
  ↓
onHtmlLoaded
  ↓
Final Website HTML
```

Example:

```html
<div class="mermaid">
graph TD
A --> B
</div>
```

or:

```html
<div class="plantuml-browser-render" data-url="..."></div>
```

These structures are suitable for browsers because JavaScript can render them.

---

## New Export Rendering Pipeline

```text
Markdown
  ↓
Parsedown
  ↓
HTML
  ↓
onHtmlLoaded
  ↓
onExportHtmlLoaded
  ↓
Export-Safe HTML
```

Example:

```html
<figure>
    <img src="/cache/generated/plantuml/hash.svg">
</figure>
```

or:

```html
<img src="/cache/generated/mermaid/hash.png">
```

The resulting HTML contains only static assets and is suitable for:

- EPUB
- PDF
- Static exports

---

# New Event

## Event Name

```text
onExportHtmlLoaded
```

The event should be dispatched only when export-safe HTML is required.

Examples:

- Ebook Plugin
- PDF Plugin
- Static Export Plugin

---

# Event Payload

The payload remains consistent with existing Typemill events.

Example:

```php
$event->setData($html);
```

Plugins receive:

```php
$html = $event->getData();
```

and return:

```php
$event->setData($html);
```

after modifications.

---

# Ebook Plugin Integration

Before rendering EPUB or PDF:

```php
$html = $page['content'];

$event = new Event();
$event->setData($html);

$dispatcher->dispatch($event, 'onExportHtmlLoaded');

$html = $event->getData();
```

The resulting HTML is then passed to:

- WeasyPrint
- EPUB generation
- other export engines

---

# PlantUML Plugin Refactor

## Remove

Remove Twig filter registration:

```php
$this->addTwigFilter('plantuml', ...);
```

Remove:

```php
processHtmlContent()
```

if it is no longer used elsewhere.

---

## Subscribe To New Event

```php
public static function getSubscribedEvents()
{
    return [
        'onHtmlLoaded'       => 'onHtmlLoaded',
        'onExportHtmlLoaded' => 'onExportHtmlLoaded',
        'onCspLoaded'        => 'onCspLoaded'
    ];
}
```

---

## Browser Rendering

Keep existing implementation:

```text
PlantUML block
  ↓
div + data attributes
  ↓
JavaScript rendering
```

---

## Export Rendering

New handler:

```php
public function onExportHtmlLoaded($event)
{
    $html = $event->getData();

    // find PlantUML blocks
    // generate static image if needed
    // replace with img tag

    $event->setData($html);
}
```

---

# Static Asset Strategy

## Recommended

Store generated assets in:

```text
/cache/generated/
```

Example:

```text
/cache/generated/plantuml/
/cache/generated/mermaid/
/cache/generated/math/
```

**Rationale:** `/cache` is publicly accessible and intended for disposable derived files, whereas `/data` is private (routed through `index.php`) and `/media` is reserved for user uploads.

---

## Asset Naming

Use content hashes:

```php
$hash = sha1($source);
```

Result:

```text
plantuml_a3d8c2.svg
```

Benefits:

- deterministic
- reusable
- no duplicates
- automatic cache invalidation

---

# Generic Helper

Implemented as protected methods in `system/typemill/Plugin.php`:

```php
protected function generateStaticAsset(
    string $source,
    callable $generator,
    string $extension = 'svg',
    string $namespace = null
): string
```

Behavior:

```text
Auto-detect plugin name as namespace
    ↓
Generate hash from source
    ↓
Check /cache/generated/{namespace}/{hash}.{ext}
    ↓
Call generator if missing
    ↓
Write file to cache
    ↓
Return public URL (with baseurl from container)
```

Example inside a plugin method:

```php
$url = $this->generateStaticAsset(
    $uml,
    function($uml) {
        return $this->renderPlantUml($uml);
    },
    'svg'
);
```

For EPUB/PDF generators that need a local filesystem path instead of a URL:

```php
$path = $this->getStaticAssetPath($uml, 'svg');
```

---

# Advantages

## Simpler Plugin Development

Plugin authors only need:

- browser renderer
- export renderer

No Twig knowledge required.

---

## Better Separation Of Concerns

Parsedown:

```text
Markdown → HTML
```

Plugins:

```text
HTML → Enhanced HTML
```

Export plugins:

```text
Enhanced HTML → Export
```

---

## Eliminates Twig Lifecycle Issues

No:

- Twig filters
- Twig functions
- Twig extension registration timing
- Twig version incompatibilities

---

## Reusable For Many Plugins

Suitable for:

- PlantUML
- Mermaid
- KaTeX
- MathJax
- Chart.js
- Graphviz
- D2
- Vega-Lite

---

# Migration Plan

## Phase 1

Add:

```text
onExportHtmlLoaded
```

to Typemill core.

---

## Phase 2

Refactor PlantUML plugin to use:

```text
onExportHtmlLoaded
```

instead of Twig filters.

---

## Phase 3

Update Ebook plugin to dispatch:

```text
onExportHtmlLoaded
```

before PDF/EPUB generation.

---

## Phase 4

Create shared asset cache helper for future plugins.

**Status:** Implemented as protected methods in `system/typemill/Plugin.php`.

---

# Result

Typemill gains a dedicated export rendering pipeline that:

- is independent of Twig
- works for EPUB and PDF generation
- supports static asset generation
- scales to future visualization plugins
- avoids runtime rendering issues and export incompatibilities

---

# Howto for Plugin Developers

This section explains how to add the static-assets export feature to your own Typemill plugin.

---

## Overview

Each plugin that renders dynamic browser content (diagrams, math, charts) now has **two rendering paths**:

| Path | Event | Output | Used By |
|------|-------|--------|---------|
| Browser | `onHtmlLoaded` | `<div>` + data-attrs + JS | Website visitors |
| Export  | `onExportHtmlLoaded` | `<img>` with static asset | EPUB, PDF, static exports |

Your plugin must implement **both** handlers. They are independent — one does not replace the other.

---

## Step 1 — Subscribe to the Events

Add both events to your plugin's `getSubscribedEvents()`:

```php
public static function getSubscribedEvents()
{
    return [
        'onHtmlLoaded'         => 'onHtmlLoaded',
        'onExportHtmlLoaded'   => 'onExportHtmlLoaded',
    ];
}
```

---

## Step 2 — Browser Rendering (`onHtmlLoaded`)

Keep your existing logic unchanged. Typical flow:

1. Parsedown produces a placeholder (e.g. `<div class="mermaid">...</div>`).
2. Your `onHtmlLoaded` listener injects the JS runtime (`addJS`).
3. The browser renders the diagram dynamically.

```php
public function onHtmlLoaded($html)
{
    $data = $html->getData();

    # inject the JS library into the page
    $this->addJS('/myplugin/public/renderer.js');

    # optional: add a data-attribute the JS will look for
    # (usually already present in the raw HTML from Parsedown)

    $html->setData($data);
}
```

No changes are required here for the export feature.

---

## Step 3 — Export Rendering (`onExportHtmlLoaded`)

This is the **new** handler. It runs only when an export plugin (Ebook, PDF, Static) asks for export-safe HTML.

### 3a — Read the payload safely

Export plugins may pass a **string** (legacy) or an **associative array** (recommended). Handle both:

```php
public function onExportHtmlLoaded($event)
{
    $data = $event->getData();

    if (is_array($data))
    {
        $html     = $data['html'];
        $item     = $data['item']     ?? [];
        $baseUrl  = $data['base_url']  ?? '';
    }
    else
    {
        $html = $data;
        $item = [];
        $baseUrl = '';
    }

    // ... your transformation logic ...

    // write back in the same format
    if (is_array($data))
    {
        $data['html'] = $html;
        $event->setData($data);
    }
    else
    {
        $event->setData($html);
    }
}
```

Why `item` and `base_url` matter:
- `item['url']` or `item['path']` let you resolve page-relative links.
- `base_url` lets you build absolute URLs if the export engine needs them.

---

### 3b — Replace dynamic markup with static images

Use `preg_replace_callback` (or DOM manipulation) to find your plugin's placeholders and swap them for `<img>` tags.

```php
$html = preg_replace_callback(
    '/<div class="myplugin">(.*?)<\/div>/s',
    function ($matches) {
        $source = trim($matches[1]);

        $url = $this->generateStaticAsset(
            $source,
            function ($src) {
                return $this->renderMyPluginSvg($src);
            },
            'svg'
        );

        return '<img src="' . $url . '" alt="Diagram" />';
    },
    $html
);
```

Rules of thumb:
- Never leave raw JS-dependent markup in export HTML.
- Always produce a single self-contained `<img>` (or `<figure>`) element.
- Use `$this->generateStaticAsset()` so you do not reinvent caching, hashing, or path logic.

---

### 3c — The generator callable

The generator receives the **raw source** and must return the **binary or text content** of the asset.

```php
function ($source) {
    // $source might be:
    //   "graph TD\nA --> B"            (mermaid)
    //   "E = mc^2"                     (math)
    //   "class Foo { ... }"             (plantuml)

    $svg = $this->callExternalRenderer($source);

    return $svg;   // must be a string (file contents)
}
```

If your renderer produces a file on disk, read it and return the contents:

```php
function ($source) {
    $tmpFile = $this->renderToTempFile($source);
    $content = file_get_contents($tmpFile);
    unlink($tmpFile);
    return $content;
}
```

---

## Step 4 — Use the Plugin Helper Methods

### Generate and get a public URL

```php
$url = $this->generateStaticAsset(
    $source,             // raw source string
    $generatorCallable,  // callable that returns file contents
    'svg'                // file extension
);
// Returns: /cache/generated/myplugin/abc123.svg
```

### Get the absolute filesystem path (for EPUB/PDF embedders)

```php
$path = $this->getStaticAssetPath($source, 'svg');
// Returns: /var/www/html/typemill/cache/generated/myplugin/abc123.svg
```

Why both methods exist:
- `generate()` creates the file **on demand** and returns a web-friendly URL for `<img src="...">`.
- `getPath()` only computes the path — useful when an export engine needs to embed the image directly into the final package.

---

## Step 5 — Testing Your Plugin

### Website test
1. Open a page that contains your plugin's Markdown syntax.
2. Verify the browser renders the diagram / formula / chart dynamically.
3. Check that `onHtmlLoaded` still behaves exactly as before.

### Export test
1. Install or use an export plugin (e.g. Ebook) that dispatches `onExportHtmlLoaded`.
2. Trigger an export (PDF or EPUB).
3. Inspect the generated HTML inside the export:
   - No `<div class="mermaid">` or `<script>` tags should remain.
   - Only `<img src="/cache/generated/...">` tags should be present.
4. Open the generated image URL directly in a browser to confirm it exists in `/cache/generated/`.

---

## Common Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| Export contains blank images | Generator callable threw an exception silently | Add `error_log()` inside the callable and check PHP error logs. |
| Same asset regenerated on every export | Source string has trailing whitespace or line-breaks | `trim()` the source before passing it to `generateStaticAsset()`. |
| EPUB/PDF cannot find images | Export engine does not resolve web URLs | Use `getStaticAssetPath()` and embed the file directly. |
| Plugin breaks on older Typemill | Event not dispatched by core | Wrap `onExportHtmlLoaded` handler in a `method_exists` check or version guard. |

---

## Cleanup

`/cache/generated/` can be deleted safely at any time. Files are recreated on demand by the next export.

---

# Reviewer Comments

## 1. Asset Storage Location

The plan suggests `/media/generated/` or `/media/.cache/`.

Per Typemill architecture (`/ai/docs/architecture.md`), `/media` is reserved for **user-uploaded media files**. Generated assets are **derived data** and should live in `/data/` or `/cache/`.

**Recommendation:** Use `/cache/generated/` instead:

```text
/cache/generated/plantuml/
/cache/generated/mermaid/
/cache/generated/math/
```

Benefits:
- Aligns with architectural rule: *"/data and /cache are always derived and may be deleted safely"*
- `/cache` is publicly accessible, so generated images can be served directly by the web server
- Keeps `/media` strictly for user uploads
- Simplifies backup and cache-invalidation strategies

---

## 2. Event Payload / Page Context

The plan proposes that `onExportHtmlLoaded` carries only the HTML string:

```php
$event->setData($html);
```

This is consistent with `onHtmlLoaded`, but export renderers may need **page context** (e.g., `$item['url']`, `$item['path']`, base URL, metadata) to:
- Resolve relative links correctly
- Generate page-specific asset paths
- Access page metadata for conditional rendering

**Recommendation:** Consider passing an associative array so listeners have access to both HTML and page context:

```php
$event->setData([
    'html' => $html,
    'item' => $item,
    'base_url' => $urlinfo['baseurl']
]);

// Listener
$data = $event->getData();
$html = $data['html'];
$item = $data['item'];
```

If backward compatibility with existing string-only events is a concern, this could be a dedicated `OnExportHtmlLoaded` event class that extends `BaseEvent` and adds a `setItem()`/`getItem()` helper.

---

## 3. URL-to-Path Mapping for EPUB/PDF Generators

The plan shows export HTML containing web-style URLs:

```html
<img src="/cache/generated/plantuml/hash.svg">
```

EPUB and PDF generators (WeasyPrint, DomPDF, etc.) typically need **local filesystem paths** or packaged assets, not web URLs.

**Recommendation:** Add a note in Phase 3 (Ebook Plugin Integration) that the Ebook plugin (or PDF plugin) is responsible for:
- Mapping these public URLs to actual file paths (`/cache/generated/...`)
- Embedding or copying the referenced assets into the final export package

Alternatively, if the event carries file paths alongside URLs, export plugins can use them directly.

---

## 4. Cache Invalidation & Cleanup

The content-hash naming strategy is excellent:

```php
$hash = sha1($source);
```

**Recommendation:** Add a note that `/cache/generated/` can be safely deleted at any time. A future helper (Phase 4) could include a simple `clear()` or garbage-collection method, but for now manual deletion or a periodic cleanup is sufficient.

---

# Verdict

The overall plan is sound, well-phased, and architecturally consistent. The above points are refinements rather than blockers.

---

# Implementation Log

## Phase 1 — Core Event

Created `system/typemill/Events/OnExportHtmlLoaded.php`:

```php
class OnExportHtmlLoaded extends BaseEvent
{

}
```

The event is available for dispatch by export plugins (Ebook, PDF, Static Export).

## Phase 4 — Asset Helper

Added protected methods to `system/typemill/Plugin.php`:
- `generateStaticAsset(...)` — generates the asset if missing and returns a public URL
- `getStaticAssetPath(...)` — returns the absolute filesystem path for embedding into EPUB/PDF

Created `/cache/generated/` directory with `drwxrwsr-x` permissions so the web server group can write generated assets.

## Corrections Applied

- **Storage path:** Changed from `/media/generated/` to `/cache/generated/` because `/cache` is publicly accessible and architecturally correct for derived disposable assets, whereas `/data` is private (routed through `index.php`) and `/media` is reserved for user uploads.
- **Event payload:** Documented recommendation that export plugins pass an associative array with `html`, `item`, and `base_url` so listeners have full page context.

---

# Examples

## Mermaid Plugin

### Website (Browser)

Parsedown turns Markdown into HTML. The plugin keeps the raw markup so the browser can render it dynamically:

```html
<div class="mermaid">
graph TD
    A --> B
</div>
```

The plugin also injects the Mermaid JS library via `addJS()` in `onHtmlLoaded` or `onTwigLoaded`.

### Export (EPUB / PDF / Static)

For exports the JS runtime is not available. The `onExportHtmlLoaded` handler replaces the `<div>` with a static image:

```php
public function onExportHtmlLoaded($event)
{
    $data = $event->getData();
    $html = is_array($data) ? $data['html'] : $data;

    $html = preg_replace_callback(
        '/<div class="mermaid">(.*?)<\/div>/s',
        function ($matches) {
            $source = trim($matches[1]);

            $url = $this->generateStaticAsset(
                $source,
                function ($src) {
                    return $this->renderMermaidSvg($src);
                },
                'svg'
            );

            return '<img src="' . $url . '" alt="Mermaid diagram" />';
        },
        $html
    );

    if (is_array($data))
    {
        $data['html'] = $html;
        $event->setData($data);
    }
    else
    {
        $event->setData($html);
    }
}
```

**Result:**

```html
<img src="/cache/generated/mermaid/abc123.svg" alt="Mermaid diagram" />
```

---

## Math Plugin (KaTeX / MathJax)

### Website (Browser)

```html
<span class="math">E = mc^2</span>
```

The plugin injects KaTeX or MathJax JS and CSS. The browser renders the formula at runtime.

### Export (EPUB / PDF / Static)

```php
public function onExportHtmlLoaded($event)
{
    $data = $event->getData();
    $html = is_array($data) ? $data['html'] : $data;

    $html = preg_replace_callback(
        '/<span class="math">(.*?)<\/span>/s',
        function ($matches) {
            $source = trim($matches[1]);

            $url = $this->generateStaticAsset(
                $source,
                function ($src) {
                    return $this->renderMathSvg($src);
                },
                'svg'
            );

            return '<img src="' . $url . '" alt="Math formula" />';
        },
        $html
    );

    if (is_array($data))
    {
        $data['html'] = $html;
        $event->setData($data);
    }
    else
    {
        $event->setData($html);
    }
}
```

**Result:**

```html
<img src="/cache/generated/math/xyz456.svg" alt="Math formula" />
```

---

## PlantUML Plugin

### Website (Browser)

```html
<div class="plantuml-browser-render" data-url="/api/plantuml/png/abc123"></div>
```

JavaScript fetches the PNG from the data-url and injects it into the div.

### Export (EPUB / PDF / Static)

```php
public function onExportHtmlLoaded($event)
{
    $data = $event->getData();
    $html = is_array($data) ? $data['html'] : $data;

    $html = preg_replace_callback(
        '/<div class="plantuml-browser-render" data-url="(.*?)"><\/div>/',
        function ($matches) {
            $source = $this->decodePlantUmlUrl($matches[1]);

            $url = $this->generateStaticAsset(
                $source,
                function ($src) {
                    return $this->renderPlantUmlSvg($src);
                },
                'svg'
            );

            return '<img src="' . $url . '" alt="PlantUML diagram" />';
        },
        $html
    );

    if (is_array($data))
    {
        $data['html'] = $html;
        $event->setData($data);
    }
    else
    {
        $event->setData($html);
    }
}
```

**Result:**

```html
<img src="/cache/generated/plantuml/a3d8c2.svg" alt="PlantUML diagram" />
```
