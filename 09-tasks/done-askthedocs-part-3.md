# Ask the Docs — Public API & Kixote Help Integration Plan

## Goal

Make the askthedocs plugin usable both as an **internal documentation widget** (current behaviour) and as an **external public API** that other systems — specifically Kixote in Typemill core — can call to provide contextual help.

The docs.typemill.net instance will run the public askthedocs endpoint. Any Typemill instance with Kixote enabled can point its Help tab at that endpoint (or any other askthedocs instance).

---

## Part A — Plugin Changes (`plugins/askthedocs`)

### 1. New settings in `askthedocs.yaml`

Add two new fields to make the public API opt-in and configurable:

```yaml
settings:
  public_api_enabled: false
  public_api_origin: '*'

forms:
  fields:
    public_api_enabled:
      type: checkbox
      label: Enable Public API
      description: 'Allow cross-origin requests to /api/v1/askthedocs/ask so external systems can use this instance as a documentation Q&A endpoint.'
    public_api_origin:
      type: text
      label: Allowed Origins (CORS)
      description: 'Comma-separated list of origins allowed to call the public API. Use * for any origin (not recommended for production).'
```

### 2. CORS headers on the public `ask` endpoint

The `/api/v1/askthedocs/ask` route already exists and requires no authentication, so it is technically public. However, it currently returns **no CORS headers**, which means browsers block cross-origin fetch requests.

**Change in `AskTheDocsController::ask()`:**

```
Before returning the JSON response:
  if public_api_enabled is true:
    add Access-Control-Allow-Origin header (value from public_api_origin or *)
    add Access-Control-Allow-Methods: POST, OPTIONS
    add Access-Control-Allow-Headers: Content-Type
```

Also add an `OPTIONS` pre-flight handler for the same route (or handle `OPTIONS` inside `ask()` by returning 204 with the headers).

### 3. Keep internal widget behaviour unchanged

- The `[askthedocs]` shortcode, frontend JS/CSS, and admin pages continue to work exactly as before.
- The public API toggle only affects CORS headers on the `ask` endpoint.
- All other admin routes (`reindex`, `status`, `summary`, `generate-summary`) remain ACL-protected.

### 4. Optional future hardening (documented, not implemented now)

- Rate-limiting (e.g. per-IP or per-session) to prevent abuse of a public endpoint.
- Simple API key check for the public endpoint if `public_api_key` is configured.
- Request signature / HMAC validation for high-trust integrations.

These are out of scope for this plan but noted because a truly open endpoint can be expensive.

---

## Part B — Typemill Core / Kixote Integration

### 1. New system setting: `help_endpoint_url`

Add to the global settings form (or Kixote settings):

```yaml
help_endpoint_url:
  type: text
  label: Help API Endpoint
  description: 'URL of the external askthedocs instance to use for the Help tab. Default: https://docs.typemill.net/api/v1/askthedocs/ask'
```

Stored in the existing settings YAML. No new files needed.

### 2. New Kixote UI tab: "Help"

In the Kixote Vue component (`/system/typemill/author/vue/kixote.js` or equivalent):

- Add a "Help" tab next to the existing prompt tabs (e.g. Write, Improve, Translate).
- When the Help tab is active:
  - The system prompt area is hidden or replaced with a short label: "Ask a question about Typemill documentation."
  - The user input textarea is shown.
  - Conversation history is maintained **per-tab** (Help history is separate from Write/Improve history).

### 3. New backend proxy endpoint: `/api/v1/kixote/help`

**Why a proxy instead of direct frontend fetch?**

- Keeps CORS simple (Kixote frontend always talks to its own Typemill backend).
- Allows the instance admin to change the endpoint URL centrally.
- Future-proof: auth tokens or signing can be added server-side later.

**`ControllerApiKixote::help(Request, Response)`:**

```
1. Parse body: { question: string, history: [{role, content}][] }
2. Validate: question non-empty, max 500 chars; history max 6 items
3. Read setting help_endpoint_url (fallback to docs.typemill.net default)
4. Build payload identical to askthedocs expected body:
   { question, history }
5. Forward POST request to help_endpoint_url using ApiCalls or Guzzle
6. If the external endpoint returns JSON with { answer, sources }, pass it through
7. If the external endpoint errors, return a friendly fallback message
8. Return JSON: { answer: string, sources: [{url, title}] }
```

The proxy should have a short timeout (e.g. 30 s) and forward HTTP error status codes transparently when possible.

### 4. Kixote frontend wiring

In the Kixote Vue app:

```
When Help tab is active and user submits:
  1. Append user bubble to local history
  2. POST /api/v1/kixote/help with { question, history }
  3. On success:
       - Append assistant bubble with answer
       - If sources exist, render small "Read more" links below the bubble
       - Push assistant turn to history
  4. On error:
       - Show error bubble with fallback text
```

### 5. Navigation injection

The Help tab is part of Kixote, not a separate page. No new admin route or navigation item outside Kixote is required. The "help item in the navigation" mentioned by the user refers to the **tab navigation inside the Kixote panel** (the existing tab bar).

If the user literally meant the **main admin sidebar**, the plan can be extended:

- Add a new sidebar item "Help" that opens Kixote with the Help tab pre-selected.
- This is a thin wrapper: `route => /tm/kixote?tab=help` or similar.

---

## Data Flow (End-to-End)

```
User (any Typemill admin)
  → Clicks Help tab in Kixote
  → Types: "How do I create a plugin?"
  → Kixote frontend POST /api/v1/kixote/help
    → Typemill backend proxy
      → POST https://docs.typemill.net/api/v1/askthedocs/ask
        → askthedocs plugin on docs.typemill.net
          → Builds documentation index (summaries.json)
          → Runs agentLoop with local AI adapter (Anthropic / OpenAI)
          → Returns { answer, sources }
      → Typemill backend returns response to Kixote frontend
    → Kixote renders answer + source links
```

---

## Files to Create / Modify

### Plugin (`plugins/askthedocs`)

| File | Action |
|------|--------|
| `askthedocs.yaml` | Add `public_api_enabled` and `public_api_origin` fields |
| `AskTheDocsController.php` | Add CORS logic to `ask()`; handle `OPTIONS` pre-flight |
| `askthedocs.php` | No changes required for this phase |

### Typemill Core (`system/typemill`)

| File | Action |
|------|--------|
| `settings/*.yaml` (or existing settings form) | Add `help_endpoint_url` field |
| `Controllers/ControllerApiKixote.php` | Add `help()` proxy method |
| `routes/api.php` (or relevant route file) | Register `POST /api/v1/kixote/help` |
| `author/vue/kixote.js` | Add Help tab, wire to new endpoint, render sources |
| `author/css/kixote.css` (if exists) | Minor styles for source links in Help tab |

---

## Verification Checklist

1. **Enable public API** on a test Typemill instance → `public_api_enabled = true`.
2. **CORS test**: From a local HTML file, run `fetch('http://test-instance/api/v1/askthedocs/ask', {method:'POST', body:JSON.stringify({question:'test',history:[]})})` → expect 200, not CORS error.
3. **Set help endpoint** in Kixote settings → point to the test instance's `/api/v1/askthedocs/ask`.
4. **Open Kixote → Help tab** → ask a question → verify answer and source links appear.
5. **Disable public API** → repeat CORS test → expect CORS error or 403.
6. **Internal widget** → add `[askthedocs]` shortcode to a page → verify it still works independently.

---

## Open Questions (to decide during implementation)

1. Should the Kixote proxy cache responses? (Probably not — answers may change as docs are updated.)
2. Should the Help tab share history with other Kixote tabs? (Recommended: no, keep Help history isolated.)
3. Should the proxy strip or rewrite source URLs? (e.g. if docs.typemill.net returns `/theme-guide`, should the proxy prepend `https://docs.typemill.net`?)
4. Should the askthedocs public endpoint expose a simple `GET /api/v1/askthedocs/health` for monitoring? (Nice-to-have.)
