# Ask the Docs — Agentic Traversal Q&A Plugin for Typemill

## Concept

A lightweight AI Q&A plugin that lets site visitors ask questions about the documentation via a chat widget embedded through a `[askthedocs]` shortcode. Instead of keyword matching against a full-text dump, the AI acts as a navigation agent: it receives only the top-level content index (with short summaries), decides which folder or page to drill into, and repeats until it has enough information to answer. This keeps context windows small, keeps the index always fresh, and needs no vector database.

**Flow:**
```
User question → Typemill → AI (root index + question) → AI picks folder
  → Typemill → AI (folder children) → AI picks page
    → Typemill → AI (page content) → AI answers
      → Answer + source links returned to user
```

Multi-turn chat history is maintained client-side and sent with each request so the AI can answer follow-up questions.

---

## Existing Infrastructure to Reuse

| What | Where | How used |
|------|-------|----------|
| AI HTTP calls | `system/typemill/Models/ApiCalls.php` | `makePostCall()` for Claude / ChatGPT |
| API keys + model names | `system/typemill/Models/Settings.php` | `getSecret('claudeKey')`, `getSecret('chatgptKey')` |
| Active AI service | `$this->settings['aiservice']`, `$this->settings['claudeModel']`, `$this->settings['chatgptModel']` | Read from system settings — no plugin-level API key needed |
| Navigation tree | `system/typemill/Models/Navigation.php` | `getFullDraftNavigation()`, `getItemForUrl()`, `flatten()` |
| Page content | `system/typemill/Models/Content.php` | `getLiveMarkdown($item)` |
| Plugin base class | `system/typemill/Plugin.php` | `getPluginSettings()`, `getSettings()`, `storePluginData()`, `getPluginData()`, `addJS()`, `addCSS()`, `getTwig()` |
| Demo plugin patterns | `plugins/demo/demo.php` | Event subscription, route registration, shortcode, admin nav |

**Critical:** Do NOT add API key to the plugin YAML. The plugin reads the system-wide AI settings that the admin already configured in Settings → AI. But admin should be able to choose AI service and model only for askthedocs from the existing service and model list.

---

## Files to Create / Modify

| File | Action |
|------|--------|
| `plugins/askthedocs/askthedocs.php` | **Create** – main plugin class |
| `plugins/askthedocs/AskTheDocsController.php` | **Create** – agentic loop + index management |
| `plugins/askthedocs/js/askthedocs.js` | **Create** – frontend chat widget (vanilla JS) |
| `plugins/askthedocs/css/askthedocs.css` | **Create** – widget styles |
| `plugins/askthedocs/templates/widget.twig` | **Create** – shortcode HTML output |
| `plugins/askthedocs/templates/admin.twig` | **Create** – admin summary editor + status page |
| `plugins/askthedocs/askthedocs.yaml` | **Modify** – replace api_key fields; add plugin-specific settings |

---

## Updated `askthedocs.yaml`

Remove `api_key` and `model` — these come from system settings. Keep and extend:

```yaml
name: Ask the Docs
version: 1.0.0
description: AI Q&A plugin for Typemill documentation using agentic content traversal.
author: Your Name
homepage: https://yourwebsite.com
license: MIT

settings:
  max_steps: 6          # Max agent loop iterations before forcing an answer
  max_pages: 3          # Max full pages fetched per question
  widget_title: 'Ask the Docs'
  auto_reindex: true    # Rebuild summary index on content changes
  system_prompt: 'You are a documentation navigation agent. You may only respond with valid JSON. Never add any text outside the JSON object.'

forms:
  fields:

    widget_title:
      type: text
      label: Widget Title
      description: 'Heading shown inside the chat widget.'

    max_steps:
      type: number
      label: Max Agent Steps
      description: 'Maximum loop iterations before the AI is forced to answer (recommended: 4–8).'

    max_pages:
      type: number
      label: Max Pages Fetched
      description: 'Maximum number of full pages the AI may read per question (recommended: 2–4).'

    system_prompt:
      type: textarea
      label: Agent System Prompt
      description: 'Instructions for the AI navigation agent. Must instruct the AI to return only valid JSON.'

    auto_reindex:
      type: checkbox
      label: Auto-reindex on content changes
      description: 'Rebuild the summary index whenever a page is published or deleted.'
```

---

## Main Plugin Class: `askthedocs.php`

**Namespace:** `Plugins\askthedocs`
**Extends:** `Typemill\Plugin`
**License:** Free (no `setPremiumLicense()` needed, or return `false`)

### Subscribed events

| Event | Handler | Purpose |
|-------|---------|---------|
| `onShortcodeFound` | `onShortcodeFound` | Render `[askthedocs]` shortcode → widget HTML |
| `onTwigLoaded` | `onTwigLoaded` | Inject JS + CSS on frontend pages only |
| `onSystemnaviLoaded` | `onSystemnaviLoaded` | Add "Ask the Docs" to admin nav |
| `onPageReady` | `onPageReady` | Render admin page content |
| `onPagePublished` | `onContentChanged` | Rebuild summary index if auto_reindex enabled |
| `onPageDeleted` | `onContentChanged` | Rebuild summary index if auto_reindex enabled |
| `onCacheUpdated` | `onContentChanged` | Rebuild summary index if auto_reindex enabled |

### Routes

```php
public static function addNewRoutes()
{
    return [
        // Public — answer a question (no auth required)
        [
            'httpMethod' => 'post',
            'route'      => '/api/v1/askthedocs/ask',
            'name'       => 'askthedocs.ask',
            'class'      => 'Plugins\askthedocs\AskTheDocsController:ask',
        ],

        // Admin — rebuild summary index
        [
            'httpMethod' => 'post',
            'route'      => '/api/v1/askthedocs/reindex',
            'name'       => 'askthedocs.reindex',
            'class'      => 'Plugins\askthedocs\AskTheDocsController:reindex',
            'resource'   => 'system',
            'privilege'  => 'view',
        ],

        // Admin — update a single page summary
        [
            'httpMethod' => 'post',
            'route'      => '/api/v1/askthedocs/summary',
            'name'       => 'askthedocs.summary',
            'class'      => 'Plugins\askthedocs\AskTheDocsController:updateSummary',
            'resource'   => 'system',
            'privilege'  => 'view',
        ],

        // Admin — get index status + summary list
        [
            'httpMethod' => 'get',
            'route'      => '/api/v1/askthedocs/status',
            'name'       => 'askthedocs.status',
            'class'      => 'Plugins\askthedocs\AskTheDocsController:status',
            'resource'   => 'system',
            'privilege'  => 'view',
        ],

        // Admin — generate AI summary for one page
        [
            'httpMethod' => 'post',
            'route'      => '/api/v1/askthedocs/generate-summary',
            'name'       => 'askthedocs.generate-summary',
            'class'      => 'Plugins\askthedocs\AskTheDocsController:generateSummary',
            'resource'   => 'system',
            'privilege'  => 'view',
        ],

        // Admin page (system UI shell)
        [
            'httpMethod' => 'get',
            'route'      => '/tm/askthedocs',
            'name'       => 'askthedocs.admin',
            'class'      => 'Typemill\Controllers\ControllerWebSystem:blankSystemPage',
            'resource'   => 'system',
            'privilege'  => 'view',
        ],
    ];
}
```

### Event handler details

**`onShortcodeFound`:**
```
if shortcode name == 'askthedocs':
  stopPropagation()
  settings = getPluginSettings()
  twig = getTwig(); loader = twig->getLoader(); loader->addPath(__DIR__ . '/templates')
  html = twig->fetch('/widget.twig', ['widget_title' => settings['widget_title']])
  shortcode->setData(html)
```

**`onTwigLoaded`:**
```
if not $this->adminroute:
  addCSS('/askthedocs/css/askthedocs.css')
  addJS('/askthedocs/js/askthedocs.js')
```

**`onSystemnaviLoaded`:**
```
navi['AskTheDocs'] = ['title' => 'Ask the Docs', 'routename' => 'askthedocs.admin',
                      'icon' => '<svg icon>', 'aclresource' => 'system', 'aclprivilege' => 'view']
if route == 'tm/askthedocs':
  navi['AskTheDocs']['active'] = true
  addJS('/askthedocs/js/admin.js')   // small inline JS for admin page
navidata->setData(navi)
```

**`onPageReady`:**
```
if adminroute && route == 'tm/askthedocs':
  twig = getTwig(); loader->addPath(__DIR__ . '/templates')
  content = twig->fetch('/admin.twig', [])
  pagedata = data->getData()
  pagedata['content'] = content
  data->setData(pagedata)
```

**`onContentChanged`:**
```
settings = getPluginSettings()
if settings['auto_reindex']:
  controller = new AskTheDocsController($this->container)
  controller->buildSummaryIndex()
```

---

## Controller: `AskTheDocsController.php`

### `ask(Request, Response)` — Public endpoint

**Request body:** `{ "question": string, "history": [{role, content}][] }`

```
1. Validate: question non-empty, max 500 chars; history max 10 items
2. Load system settings: aiservice, model, api key via Settings::getSecret()
3. If aiservice == 'none' or no API key → return 400 "AI service not configured"
4. Load summaries from getPluginData('summaries.json')
5. Build root index from Navigation->getFullDraftNavigation():
   map top-level items → IndexNode[]
   attach summaries from summaries.json
6. Run agentLoop(question, rootIndex, history, settings)
7. Return { answer: string, sources: [{url, title}] }
```

### `agentLoop(question, rootIndex, history, settings)` — Private

```
$context = []           // accumulated IndexNode data
$pagesRead = 0
$step = 0
$maxSteps = settings['max_steps'] (default 6)
$maxPages = settings['max_pages'] (default 3)
$messages = [...history (last 6 turns)]

while ($step < $maxSteps):
  $step++

  // Build message for this step
  if $step == 1:
    userContent = question + "\n\nDocumentation index:\n" + json_encode(rootIndex)
  else:
    userContent = json_encode(lastStepData)  // result of last action

  $messages[] = ['role' => 'user', 'content' => $userContent]

  // If at limit, force answer
  if $step >= $maxSteps or $pagesRead >= $maxPages:
    $messages[] = ['role' => 'user', 'content' => '{"action":"force_answer"}']

  // Call AI
  $raw = callAI($messages, $settings)

  // Parse JSON; retry once on invalid JSON
  $agentResponse = json_decode($raw, true)
  if JSON invalid:
    retry callAI once with appended "return valid JSON only"
    if still invalid: break with error

  $messages[] = ['role' => 'assistant', 'content' => $raw]

  if $agentResponse['action'] == 'open_folder':
    path = $agentResponse['path']
    children = getFolder(path)   // load from Navigation model
    attach summaries to children
    $lastStepData = ['path' => path, 'children' => children]
    $context[] = $lastStepData

  elseif $agentResponse['action'] == 'open_page':
    path = $agentResponse['path']
    $pagesRead++
    content = getPage(path)      // load from Content model
    $lastStepData = ['path' => path, 'content' => content]
    $context[] = $lastStepData
    $sources[] = path

  elseif $agentResponse['action'] == 'answer':
    return ['answer' => $agentResponse['answer'], 'sources' => $sources]

// If loop ends without answer:
return ['answer' => 'Could not find an answer in the documentation.', 'sources' => []]
```

### `callAI(messages, settings)` — Private

Uses the system AI settings — no plugin-level API key:

```php
private function callAI(array $messages, array $settings): string
{
    $aiservice  = $settings['aiservice'] ?? false;
    $settingsModel = new Settings();
    $apiCalls   = new ApiCalls();
    $apiCalls->setTimeout(60);

    if ($aiservice === 'claude') {
        $apikey = $settingsModel->getSecret('claudeKey');
        $model  = $settings['claudeModel'] ?? 'claude-3-5-haiku-20241022';
        $url    = 'https://api.anthropic.com/v1/messages';
        $headers = ["x-api-key: $apikey", "anthropic-version: 2023-06-01"];
        $payload = [
            'model'      => $model,
            'system'     => $settings['plugins']['askthedocs']['system_prompt'],
            'messages'   => $messages,
            'max_tokens' => 512,
        ];
        $raw = $apiCalls->makePostCall($url, $payload, $headers);
        $data = json_decode($raw, true);
        return $data['content'][0]['text'] ?? '';
    }

    if ($aiservice === 'chatgpt') {
        $apikey = $settingsModel->getSecret('chatgptKey');
        $model  = $settings['chatgptModel'] ?? 'gpt-4o-mini';
        $url    = 'https://api.openai.com/v1/chat/completions';
        $headers = "Authorization: Bearer $apikey";
        // prepend system message for ChatGPT
        $chatMessages = array_merge(
            [['role' => 'system', 'content' => $settings['plugins']['askthedocs']['system_prompt']]],
            $messages
        );
        $payload = ['model' => $model, 'messages' => $chatMessages, 'max_tokens' => 512];
        $raw = $apiCalls->makePostCall($url, $payload, $headers);
        $data = json_decode($raw, true);
        return $data['choices'][0]['message']['content'] ?? '';
    }

    return '';
}
```

### `getFolder(path)` — Private

```
1. $nav = Navigation->getFullDraftNavigation($urlinfo, false)
2. $item = Navigation->getItemForUrl($path, $urlinfo, false)
3. if no item → return []
4. children = $item->folderContent (array of item objects)
5. return mapped IndexNode[]:
   [ {type, title, summary (from summaries.json), path: item->urlRelWoF} ]
```

### `getPage(path)` — Private

```
1. $item = Navigation->getItemForUrl($path, $urlinfo, false)
2. if no item → return ''
3. $markdown = Content->getLiveMarkdown($item)
4. return $markdown
```

### `buildSummaryIndex()` — Public (called from event hook + reindex route)

```
1. $nav = Navigation->getFullDraftNavigation($urlinfo, false)
2. $flat = Navigation->flatten($nav, '', [])
3. $summaries = load existing getPluginData('summaries.json') or []
4. For each item in flat where elementType == 'file' && status == 'published':
   url = item->urlRelWoF
   if url not in existing summaries:
     // Use meta description as default summary
     $meta = Meta->getMetaData($item)
     $summary = $meta['meta']['description'] ?? ''
     $summaries[$url] = ['title' => item->name, 'summary' => $summary]
5. storePluginData('summaries.json', json_encode($summaries))
6. return count($summaries)
```

### `generateSummary(Request, Response)` — Admin: AI-generate one page's summary

```
Request: { "path": "/some/page" }
1. Load page markdown via Content->getLiveMarkdown()
2. Truncate to ~2000 chars
3. Call AI (callAI) with prompt: "Summarize this page in 1-2 sentences for a navigation index."
4. Store result in summaries.json for this path
5. Return { "summary": "..." }
```

### `reindex(Request, Response)` — Admin

```
1. buildSummaryIndex()
2. Return { pages: N, message: "Index rebuilt." }
```

### `status(Request, Response)` — Admin

```
1. Load summaries.json
2. Return { built: timestamp, pagecount: N, summaries: [{url, title, summary}] }
```

### `updateSummary(Request, Response)` — Admin

```
Request: { "path": "/some/page", "summary": "..." }
1. Load summaries.json
2. Update entry for path
3. storePluginData('summaries.json', ...)
4. Return { message: "Saved." }
```

---

## Agent Protocol

### System prompt (default, configurable in plugin settings)

```
You are a documentation navigation agent.
You may only respond with valid JSON.
Actions: open_folder, open_page, answer
Never explain. Never add text outside the JSON object.
```

### Request format sent to AI at each step

```json
{
  "action": "open_folder" | "open_page" | "answer",
  "path": "/url-path",      // for open_folder and open_page
  "answer": "string",       // for answer action
  "confidence": 0.0–1.0     // optional
}
```

### IndexNode (sent to AI)

```json
{
  "type": "folder" | "page",
  "title": "string",
  "summary": "string",
  "path": "/url-path"
}
```

### Protocol rules (enforced server-side)

- Agent must choose exactly one action per response
- `path` must exist in the last received index (validated before use)
- `answer` action only valid if AI has seen at least one page or is at step limit
- If step >= maxSteps → inject `{"action":"force_answer"}` and require answer
- If page count >= maxPages → stop opening pages, allow answer only
- Invalid JSON → retry once with clarification message; if still invalid → abort with fallback answer
- Ignore unknown actions (treat as no-op, continue loop)

---

## Data Storage

| File | Location | Content |
|------|----------|---------|
| `summaries.json` | `data/askthedocs/summaries.json` | URL-keyed map of page summaries |

### `summaries.json` structure

```json
{
  "built": "2026-02-19T12:00:00+00:00",
  "/theme-guide": {
    "title": "Theme Guide",
    "summary": "Explains how to build and configure Typemill themes."
  },
  "/api/authentication": {
    "title": "Authentication",
    "summary": "Describes API token usage and login flow."
  }
}
```

Summaries come from:
1. **Meta description** (auto-populated when index is built)
2. **AI-generated** via admin "Generate summary" button
3. **Manually edited** by admin in the summary editor

---

## Frontend Chat Widget

### `templates/widget.twig`

```html
<div class="atd-widget"
     id="atd-widget"
     data-endpoint="{{ base_url() }}/api/v1/askthedocs/ask">
  <h3 class="atd-title">{{ widget_title }}</h3>
  <div class="atd-messages" id="atd-messages" aria-live="polite"></div>
  <form class="atd-form" id="atd-form" novalidate>
    <textarea id="atd-input" placeholder="Ask a question…" rows="2" maxlength="500"></textarea>
    <button type="submit">Ask</button>
  </form>
</div>
```

### `js/askthedocs.js` — Vanilla JS, no framework

```
State: history = []   // [{role, content}], capped at 6 items

On DOMContentLoaded:
  find #atd-widget, read data-endpoint
  attach submit to #atd-form

On submit:
  prevent default
  question = #atd-input value, trimmed, validate non-empty
  append user bubble to #atd-messages
  push {role:'user', content: question} to history
  clear input, disable submit, show "Thinking…" bubble

  fetch(endpoint, {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({ question, history: history.slice(-6) })
  })
  .then(res => res.json())
  .then(data => {
    remove "Thinking…" bubble
    append assistant bubble with data.answer (HTML-escaped)
    if data.sources && data.sources.length:
      append "<div class='atd-sources'>Sources: <a href=...>title</a>, ...</div>"
    push {role:'assistant', content: data.answer} to history
    scroll #atd-messages to bottom
    re-enable submit
  })
  .catch(() => show "An error occurred, please try again." bubble)
```

### `css/askthedocs.css` — CSS custom properties (theme-overridable)

```css
:root {
  --atd-bg: #f9f9f9;
  --atd-border: #ddd;
  --atd-user-bg: #0070f3;
  --atd-user-color: #fff;
  --atd-bot-bg: #eee;
  --atd-bot-color: #333;
}
/* .atd-widget, .atd-messages, .atd-bubble.user, .atd-bubble.assistant, .atd-sources, .atd-form */
```

---

## Admin Page: `templates/admin.twig`

Shows:
- **Index status**: last built timestamp + page count
- **"Rebuild Index"** button → POST `/api/v1/askthedocs/reindex`
- **Summary editor table**: URL | Title | Summary (editable textarea) | "Generate with AI" button per row
- Small inline JS (`admin.js`) wires up all buttons via `fetch()`

---

## Verification Checklist

1. **Configure AI** in Typemill admin → Settings → AI: select Claude or ChatGPT, enter API key, save
2. **Activate plugin** in Settings → Plugins → Ask the Docs
3. **Build summary index** at `/tm/askthedocs` → confirm page count and review summaries
4. **Optionally edit summaries** or click "Generate with AI" for better descriptions
5. **Add shortcode** `[askthedocs]` to any content page
6. **Visit page** as anonymous visitor → widget renders
7. **Ask a question** → AI traverses the index and returns an answer with source links
8. **Ask a follow-up** → previous turn is preserved in context
9. **Publish a new page** → summary index auto-rebuilds (check status reflects new count)
10. **Test edge cases**: empty question (rejected client-side), AI not configured (400 error shown), invalid AI JSON response (retried once then fallback answer)

---

## Further Improvements (future)

- Batch summary generation for all pages via admin button
- Fetch multiple index levels at once to reduce loop steps
- Confidence threshold: if AI confidence < 0.5, go deeper before answering
- Per-page `[atd-instructions]` shortcode to inject custom hints into the index node
- Support for Kixote (self-hosted) when available
