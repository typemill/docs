# Plugin Developer Guide: Export-Safe Static Assets

## Introduction

Typemill now provides a dedicated export rendering pipeline that lets plugins generate static assets (SVG, PNG) for export formats like EPUB, PDF, and static site exports. This means plugins that render dynamic browser content — such as Mermaid diagrams, PlantUML charts, or math formulas — can also provide a static, export-safe version.

This guide explains how to add the export feature to your plugin.

---

## Architecture Overview

Typemill has **two rendering pipelines**:

### Browser Pipeline (existing)

```text
Markdown
  → Parsedown
    → HTML
      → onHtmlLoaded
        → Website (browser renders with JS)
```

Plugins inject JavaScript libraries and use `<div>` placeholders with data attributes. The browser renders diagrams dynamically at runtime.

### Export Pipeline (new)

```text
Markdown
  → Parsedown
    → HTML
      → onHtmlLoaded
        → onExportHtmlLoaded
          → Export-Safe HTML (static <img> tags only)
```

Export plugins (Ebook, PDF, Static Export) dispatch `onExportHtmlLoaded` before generating the final output. Your plugin receives the HTML, replaces dynamic placeholders with static `<img>` tags, and writes the assets to `/cache/generated/`.

---

## The Two Events

| Event | When | Purpose |
|-------|------|---------|
| `onHtmlLoaded` | Every page request | Browser rendering (divs + JS) |
| `onExportHtmlLoaded` | Only during export | Static rendering (img tags) |

Both events use the same `BaseEvent` class. The payload is the HTML string (or an associative array with `html`, `item`, and `base_url`).

---

## Quick Start

### 1. Subscribe to Both Events

```php
public static function getSubscribedEvents()
{
    return [
        'onHtmlLoaded'       => 'onHtmlLoaded',
        'onExportHtmlLoaded' => 'onExportHtmlLoaded',
    ];
}
```

### 2. Keep Browser Rendering Unchanged

```php
public function onHtmlLoaded($html)
{
    $data = $html->getData();

    // Inject JS library for browser rendering
    $this->addJS('/myplugin/public/renderer.js');

    $html->setData($data);
}
```

### 3. Add Export Rendering

```php
public function onExportHtmlLoaded($event)
{
    $data = $event->getData();

    // Handle both string and array payloads
    if (is_array($data))
    {
        $html    = $data['html'];
        $item    = $data['item']    ?? [];
        $baseUrl = $data['base_url'] ?? '';
    }
    else
    {
        $html = $data;
    }

    // Replace dynamic markup with static images
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

    // Write back in the same format
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

---

## Plugin Base Class Helpers

Typemill provides two protected methods in the `Plugin` base class for generating static assets.

### Generate an Asset and Get a URL

```php
$url = $this->generateStaticAsset(
    $source,                // raw source string
    function ($src) {       // generator callable
        return $this->renderMermaidSvg($src);
    },
    'svg'                   // file extension
);
// Returns: /cache/generated/mermaid/abc123.svg
```

How it works:
1. Uses the plugin name automatically as the namespace
2. Hashes the source with `sha1()`
3. Checks if `/cache/generated/{namespace}/{hash}.{ext}` already exists
4. Calls the generator callable only if the file is missing
5. Returns the public URL (with base URL from the container)

---

## What a Generator Callable Looks Like

The generator callable receives the raw source string and must return the **file contents** (binary or text). Here is a conceptual example:

```php
private function renderMathSvg(string $latex): string
{
    // 1. Sanitize or validate the input
    $latex = trim($latex);

    // 2. Call an external tool or library
    //    Option A: Command-line tool (e.g. dvisvgm, katex-cli, mmdc)
    $command = escapeshellcmd("math-cli --format svg --input " . escapeshellarg($latex));
    $svg = shell_exec($command);

    //    Option B: PHP library (e.g. Gregwar\Tex2png, a local KaTeX wrapper)
    //    $renderer = new \MyVendor\MathRenderer();
    //    $svg = $renderer->toSvg($latex);

    //    Option C: HTTP API (e.g. self-hosted rendering service)
    //    $response = $this->httpClient->post('http://localhost:3000/render', ['body' => $latex]);
    //    $svg = $response->getBody();

    // 3. Handle errors gracefully
    if (empty($svg))
    {
        // Return a fallback SVG or throw an exception
        return '<svg><text x="10" y="20">Render error</text></svg>';
    }

    // 4. Return the raw file contents
    return $svg;
}
```

Key points:
- The callable **must return a string** containing the final file contents.
- It **must not** write to `/cache/generated/` itself — `generateStaticAsset()` handles that.
- Always sanitize inputs (especially when passing to `shell_exec`) and provide a fallback so a single broken diagram does not crash the entire export.

### Get the Filesystem Path

For EPUB/PDF embedders that need local file paths:

```php
$path = $this->getStaticAssetPath($source, 'svg');
// Returns: /var/www/html/typemill/cache/generated/mermaid/abc123.svg
```

---

## Plugin Examples

### Mermaid Plugin

**Browser (existing):**

Parsedown produces:

```html
<div class="mermaid">
graph TD
    A --> B
</div>
```

The plugin injects `mermaid.min.js` via `onHtmlLoaded`. The browser renders the diagram.

**Export (new):**

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

    is_array($data) ? $event->setData(['html' => $html] + $data) : $event->setData($html);
}
```

**Result:**

```html
<img src="/cache/generated/mermaid/abc123.svg" alt="Mermaid diagram" />
```

---

### PlantUML Plugin

**Browser (existing):**

```html
<div class="plantuml-browser-render" data-url="/api/plantuml/png/abc123"></div>
```

JavaScript fetches the PNG and injects it.

**Export (new):**

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

    is_array($data) ? $event->setData(['html' => $html] + $data) : $event->setData($html);
}
```

**Result:**

```html
<img src="/cache/generated/plantuml/a3d8c2.svg" alt="PlantUML diagram" />
```

---

### Math Plugin (KaTeX / MathJax)

**Browser (existing):**

```html
<span class="math">E = mc^2</span>
```

The plugin injects KaTeX or MathJax JS and CSS.

**Export (new):**

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

    is_array($data) ? $event->setData(['html' => $html] + $data) : $event->setData($html);
}
```

**Result:**

```html
<img src="/cache/generated/math/xyz456.svg" alt="Math formula" />
```

---

## Important Notes

- **Generator callable** must return the **file contents as a string** (binary or text). If your external tool writes to a temp file, read it with `file_get_contents()`, then return the contents.
- **Trim source strings** before passing them to `generateStaticAsset()` to avoid hash mismatches caused by trailing whitespace.
- **Backward compatibility** — always handle both string and array payloads. Older Typemill versions or export plugins may dispatch the event with a plain HTML string.
- **Cache is disposable** — `/cache/generated/` can be safely deleted at any time. Files are recreated on the next export.

---

## Files Reference

| File | Purpose |
|------|---------|
| `system/typemill/Events/OnExportHtmlLoaded.php` | The new event class |
| `system/typemill/Plugin.php` | `generateStaticAsset()` and `getStaticAssetPath()` methods |
| `/cache/generated/` | Directory where assets are stored (auto-created) |
