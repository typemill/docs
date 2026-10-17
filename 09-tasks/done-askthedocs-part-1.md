# Plan: askthedocs Plugin for Typemill

## Context

Build a complete AI-powered Q&A plugin for Typemill CMS. Visitors embed a chat widget via `[askthedocs]` shortcode on any documentation page. When a question is asked, the plugin traverses Typemill's content index to find the most relevant pages, sends them as context to the Anthropic Claude API, and returns an answer. Conversations are multi-turn (history maintained client-side). Access is public (anonymous visitors).

A skeleton YAML config already exists at `plugins/askthedocs/askthedocs.yaml`. Everything else must be created.

---

## Files to Create / Modify

| File | Action |
|------|--------|
| `plugins/askthedocs/askthedocs.php` | **Create** – main plugin class |
| `plugins/askthedocs/AskTheDocsController.php` | **Create** – API handlers |
| `plugins/askthedocs/js/askthedocs.js` | **Create** – frontend chat widget (vanilla JS) |
| `plugins/askthedocs/css/askthedocs.css` | **Create** – widget styles |
| `plugins/askthedocs/templates/widget.twig` | **Create** – shortcode HTML output |
| `plugins/askthedocs/templates/admin.twig` | **Create** – admin index management page |
| `plugins/askthedocs/askthedocs.yaml` | **Modify** – add new settings fields |

### Existing references to reuse
- **Plugin base class:** `system/typemill/Plugin.php` — `getPluginSettings()`, `storePluginData()`, `getPluginData()`, `addJS()`, `addCSS()`, `getTwig()`, `generateForm()`
- **Navigation model:** `system/typemill/Models/Navigation.php` — `getFullDraftNavigation()`, `flatten()`
- **Content model:** `system/typemill/Models/Content.php` — `getLiveMarkdown($item)`
- **Demo plugin patterns:** `plugins/demo/demo.php` — event subscription, route registration, shortcode handling, admin nav injection

---

## Implementation Plan

### Step 1 – Update `askthedocs.yaml`

Add the following fields to the existing YAML:

```yaml
settings:
  api_key: ''
  model: 'claude-sonnet-4-6'
  max_tokens: 1024
  context_pages: 5
  widget_title: 'Ask the Docs'
  auto_reindex: true
  system_prompt: 'You are a helpful assistant that answers questions about the documentation. Use only the provided context to answer. If the answer is not in the context, say so.'

forms:
  fields:
    api_key:           # existing
    model:             # existing
    max_tokens:        # existing
    system_prompt:     # existing
    context_pages:
      type: number
      label: Context Pages (1-10)
      description: 'Number of best-matching pages to inject as AI context.'
    widget_title:
      type: text
      label: Widget Title
      description: 'Heading shown inside the chat widget.'
    auto_reindex:
      type: checkbox
      label: Auto-reindex on content changes
      description: 'Rebuild the content index whenever a page is published or deleted.'
```

---

### Step 2 – Create `askthedocs.php` (main plugin class)

**Namespace:** `Plugins\askthedocs`
**Extends:** `Typemill\Plugin`

#### Subscribed events

| Event | Handler | Purpose |
|-------|---------|---------|
| `onShortcodeFound` | `onShortcodeFound` | Render `[askthedocs]` → widget HTML |
| `onTwigLoaded` | `onTwigLoaded` | Inject JS + CSS on frontend pages |
| `onSystemnaviLoaded` | `onSystemnaviLoaded` | Add admin menu item |
| `onPageReady` | `onPageReady` | Render admin page |
| `onPagePublished` | `onContentChanged` | Trigger index rebuild if auto_reindex |
| `onPageDeleted` | `onContentChanged` | Trigger index rebuild if auto_reindex |
| `onCacheUpdated` | `onContentChanged` | Trigger index rebuild if auto_reindex |

#### Routes

```php
public static function addNewRoutes()
{
    return [
        // Public: answer a question (no auth)
        ['httpMethod' => 'post', 'route' => '/api/v1/askthedocs/ask',
         'name' => 'askthedocs.ask', 'class' => 'Plugins\askthedocs\AskTheDocsController:ask'],

        // Admin: rebuild index
        ['httpMethod' => 'post', 'route' => '/api/v1/askthedocs/reindex',
         'name' => 'askthedocs.reindex', 'class' => 'Plugins\askthedocs\AskTheDocsController:reindex',
         'resource' => 'system', 'privilege' => 'view'],

        // Admin: get index status
        ['httpMethod' => 'get', 'route' => '/api/v1/askthedocs/status',
         'name' => 'askthedocs.status', 'class' => 'Plugins\askthedocs\AskTheDocsController:status',
         'resource' => 'system', 'privilege' => 'view'],

        // Admin page
        ['httpMethod' => 'get', 'route' => '/tm/askthedocs',
         'name' => 'askthedocs.admin', 'class' => 'Typemill\Controllers\ControllerWebSystem:blankSystemPage',
         'resource' => 'system', 'privilege' => 'view'],
    ];
}
```

#### `onShortcodeFound` logic
```
if shortcode name == 'askthedocs':
  stopPropagation()
  load templates/widget.twig via getTwig()
  pass settings (widget_title) to template
  setData(rendered HTML)
```

#### `onTwigLoaded` logic
```
if not $this->adminroute:
  addCSS('/askthedocs/css/askthedocs.css')
  addJS('/askthedocs/js/askthedocs.js')
```

#### `onSystemnaviLoaded` logic
```
Add menu item 'Ask the Docs' → route 'askthedocs.admin'
If current route == 'tm/askthedocs':
  mark active
```

#### `onContentChanged` logic
```
settings = getPluginSettings()
if settings['auto_reindex']:
  (new AskTheDocsController($this->container))->buildIndex()
```

---

### Step 3 – Create `AskTheDocsController.php`

#### `ask(Request $request, Response $response)`

```
1. Parse body: { question: string, history: [{role, content}][] }
2. Validate: question must be non-empty string, max 500 chars
3. Load index from getPluginData('index.json') → json_decode
4. If index missing: return 503 "Index not built yet"
5. scorePages(question, index['pages']) → top N pages (N = context_pages setting)
6. For each top page:
   - instantiate Navigation model, call getItemForUrl(url)
   - instantiate Content model, call getLiveMarkdown(item)
7. Build context string: "### {title}\nURL: {url}\n\n{markdown}\n\n---\n"
8. Build Anthropic API payload:
   - system: settings[system_prompt] + "\n\nRelevant documentation:\n\n" + context
   - messages: [...history (last 10 turns), {role:'user', content: question}]
   - model, max_tokens from settings
9. Call Anthropic API via cURL:
   POST https://api.anthropic.com/v1/messages
   Headers: x-api-key, anthropic-version: 2023-06-01, content-type: application/json
10. Parse response, extract text content
11. Return JSON: { answer: string, sources: [{url, title}] }
```

#### `reindex(Request $request, Response $response)`

```
1. Call buildIndex()
2. Return { pages: N, message: "Index built successfully." }
```

#### `status(Request $request, Response $response)`

```
1. Load index from getPluginData('index.json')
2. Return { built: timestamp, pagecount: N } or { built: null, pagecount: 0 }
```

#### `buildIndex()` (public method, called from plugin + reindex route)

```
1. Instantiate Navigation model from DI container
2. $urlinfo = $this->urlinfo()  [called on plugin] or passed in
3. $nav = Navigation->getFullDraftNavigation($urlinfo, false)
4. $flat = Navigation->flatten($nav, '', [])
5. $pages = []
6. For each item in flat:
   if item->elementType == 'file' && item->status == 'published':
     $content = Content->getLiveMarkdown($item)
     $text = stripMarkdown($content)   — regex strip of markdown syntax
     $pages[] = ['url' => item->urlRelWoF, 'title' => item->name, 'text' => $text]
7. $index = ['built' => date('c'), 'pagecount' => count($pages), 'pages' => $pages]
8. storePluginData('index.json', json_encode($index))
9. return count($pages)
```

#### `scorePages(string $question, array $pages)` (private)

```
1. Tokenize question: strtolower, preg_split on non-word chars, filter stopwords
2. For each page:
   $words = str_word_count(strtolower($page['text']), 1)
   $score = count(array_intersect($tokens, $words)) / sqrt(max(count($words), 1))
3. Sort descending by score
4. Return top N (context_pages setting, default 5)
```

Stopwords: inline PHP array of ~50 common English words (the, a, is, in, of, to, and, …).

---

### Step 4 – Create `templates/widget.twig`

```html
<div class="atd-widget"
     id="atd-widget"
     data-endpoint="{{ base_url() }}/api/v1/askthedocs/ask">
  <h3 class="atd-title">{{ widget_title }}</h3>
  <div class="atd-messages" id="atd-messages" aria-live="polite"></div>
  <form class="atd-form" id="atd-form" novalidate>
    <textarea id="atd-input"
              placeholder="Ask a question…"
              rows="2"
              maxlength="500"></textarea>
    <button type="submit">Send</button>
  </form>
</div>
```

The `data-endpoint` attribute lets `askthedocs.js` locate the correct API URL without hardcoding.

---

### Step 5 – Create `js/askthedocs.js` (vanilla JS)

```
State: history = []  (array of {role, content}, capped at 10 items)

On DOMContentLoaded:
  find #atd-widget, read data-endpoint
  attach submit handler to #atd-form

On submit:
  prevent default
  read question from #atd-input, trim, validate non-empty
  append user bubble to #atd-messages
  push {role:'user', content: question} to history
  clear input, disable button, show "Thinking…" bubble

  fetch(endpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ question, history: history.slice(-10) })
  })
  .then(res => res.json())
  .then(data => {
    remove "Thinking…" bubble
    append assistant bubble with data.answer (escaped HTML)
    if data.sources.length:
      append sources list as small links
    push {role:'assistant', content: data.answer} to history
    scroll #atd-messages to bottom
    re-enable button
  })
  .catch(err => show error bubble)
```

No external dependencies. Pure `fetch()` + DOM API.

---

### Step 6 – Create `css/askthedocs.css`

Minimal styles using CSS custom properties (overridable by themes):

```css
:root {
  --atd-bg: #f9f9f9;
  --atd-border: #ddd;
  --atd-user-bg: #0070f3;
  --atd-user-color: #fff;
  --atd-bot-bg: #eee;
  --atd-bot-color: #333;
}

.atd-widget { … }          /* container */
.atd-messages { … }        /* scrollable message area */
.atd-bubble { … }          /* shared bubble styles */
.atd-bubble.user { … }     /* right-aligned, user colour */
.atd-bubble.assistant { … }/* left-aligned, bot colour */
.atd-sources { … }         /* small source links */
.atd-form { … }            /* textarea + button row */
```

---

### Step 7 – Create `templates/admin.twig`

Admin page (loaded via `onPageReady` when on `/tm/askthedocs`):

- Index status card: last built timestamp + page count (fetched from `/api/v1/askthedocs/status`)
- "Rebuild Index" button → POST to `/api/v1/askthedocs/reindex`, show result
- Scrollable table of indexed pages (title + URL)
- Small inline JS (no full Vue app needed) to wire up the button via `fetch()`

---

## Data Storage

| File | Location | Content |
|------|----------|---------|
| `index.json` | `data/askthedocs/index.json` | Traversal index (JSON) |

Written via `$this->storePluginData('index.json', $json)`, read via `$this->getPluginData('index.json')`.

---

## Index JSON Structure

```json
{
  "built": "2026-02-19T12:00:00+00:00",
  "pagecount": 42,
  "pages": [
    {
      "url": "/theme-guide",
      "title": "Theme Guide",
      "text": "plain text stripped of markdown syntax…"
    }
  ]
}
```

---

## Verification Checklist

1. Activate plugin in Typemill admin → Settings → Plugins → Ask the Docs
2. Enter Anthropic API key and save settings
3. Build index via admin panel at `/tm/askthedocs` → confirm page count
4. Add shortcode `[askthedocs]` to any content page
5. Visit page as anonymous visitor → widget renders correctly
6. Ask a question → AI returns answer with source page links
7. Ask a follow-up → previous turn is included in the AI context
8. Publish a new page → index auto-rebuilds (check admin status reflects new count)
9. Test edge cases: empty question (client-side rejected), no index built (503 response shown)
