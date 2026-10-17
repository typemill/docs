# TODO: Extend AI adapter for OpenAI Responses API

## Problem

Typemill's generic AI integration uses the `OpenAiAdapter` in `/system/typemill/Models/AiAdapter.php`. This adapter is hard-wired to the legacy chat-completions endpoint:

```
POST {baseUrl}/chat/completions
```

Modern providers such as **OpenCode Zen** route many models through the newer **OpenAI Responses API** endpoint:

```
POST {baseUrl}/responses
```

Examples of models that require the `/responses` endpoint on OpenCode Zen include:

- `gpt-5.4-mini`
- `gpt-5.4`
- `gpt-5.5`
- `gpt-5`
- `grok-4.5`
- other GPT 5.x / Grok / Muse Spark models

When a user configures one of these models with Typemill's "openai compatible" adapter, Typemill still sends the request to `{baseUrl}/chat/completions`. The provider cannot process the model on that endpoint and returns an **HTTP 500 Internal Server Error**. Typemill then surfaces that raw error to the user, which is confusing because all settings appear to be correct.

## Root cause

1. The adapter has a single endpoint strategy (`/chat/completions`) and no way to select `/responses`.
2. The Responses API request/response shape differs from chat completions:
   - Chat completions: `{ model, messages, temperature, max_tokens, stream }`
   - Responses API: `{ model, input, temperature, max_output_tokens, ... }` where `input` can be a string or array of message objects.
3. Error handling only checks `data.error.message`; it does not translate provider-specific HTTP 500s into actionable messages.

## Impact

- Users cannot use OpenCode Zen's GPT 5.x models with Kixote.
- The error message "Internal Server Error" gives no indication that the endpoint is wrong.
- The model list endpoint (`GET {baseUrl}/models`) works, so the UI looks configured correctly even though chat calls fail.

## Proposed solution

Add a new provider adapter type, e.g. `openai-responses`, alongside the existing `openai` and `anthropic` adapters. Keep the change localized to `/system/typemill/Models/AiAdapter.php` and the AI settings form.

### 1. New `OpenAiResponsesAdapter` class

Implement `AiAdapterInterface` with the following behavior:

- Endpoint: `POST {baseUrl}/responses`
- Request body:
  ```json
  {
    "model": "gpt-5.4-mini",
    "input": [
      { "role": "system", "content": "..." },
      { "role": "user", "content": "..." }
    ],
    "temperature": 0.7,
    "max_output_tokens": 4000
  }
  ```
- Response extraction: read `output` array and find the first item with `type: "message"`, then return `content[0].text`.
- Error extraction: check `error.message`, `error.type`, and HTTP status. If the provider returns HTTP 500, return a clear message such as:
  > "The provider does not serve model `gpt-5.4-mini` on the chat-completions endpoint. Please switch to the 'OpenAI Responses API' adapter or choose a model that supports /chat/completions."

### 2. Adapter factory update

Extend `AiAdapter::create()` to support:

```php
return match ($adapter) {
    'anthropic'          => new AnthropicAdapter(...),
    'openai-responses'   => new OpenAiResponsesAdapter(...),
    default              => new OpenAiAdapter(...),
};
```

### 3. Settings form

Add `openai-responses` as an option in the AI adapter dropdown. The base URL and model fields stay the same; only the adapter choice changes.

### 4. Better error messages for the existing chat adapter

Improve `OpenAiAdapter::chat()` so that when the provider returns HTTP 500 or an empty response, the error message hints at the possible endpoint mismatch:

> "Provider returned an internal error. If you are using a model such as gpt-5.x on OpenCode Zen, try the OpenAI Responses API adapter instead."

## Acceptance criteria

- [ ] New `openai-responses` adapter exists and implements `AiAdapterInterface`.
- [ ] Adapter factory creates the correct implementation based on the configured adapter.
- [ ] Settings form exposes the new adapter option.
- [ ] Kixote prompts work with `gpt-5.4-mini` on OpenCode Zen when `openai-responses` is selected.
- [ ] `listModels()` still works for the new adapter (`GET {baseUrl}/models`).
- [ ] Error messages explain *why* a chat-completions call failed and point the user to the responses adapter.
- [ ] No changes outside `/system/typemill/Models/AiAdapter.php` and the AI settings form unless absolutely necessary.

## Notes

- This is a core capability extension, so it belongs in `/system/typemill/Models/AiAdapter.php` (allowed for adapter fixes/extensions).
- Keep the existing `OpenAiAdapter` unchanged for backward compatibility with OpenAI, Ollama, LM Studio, LocalAI, vLLM, OpenRouter, etc.
- The Responses API is intentionally close to chat completions; reuse `ApiCalls` for transport and only transform payload/response shapes.

---

## Review & Improvements

### Current state assessment

- `OpenAiAdapter` in `/system/typemill/Models/AiAdapter.php` is hard-wired to `POST {baseUrl}/chat/completions`.
- `AiAdapter::create()` currently maps `anthropic` → `AnthropicAdapter` and everything else → `OpenAiAdapter`.
- The AI settings form (`/system/typemill/settings/system.yaml`) exposes `ai_adapter` with options `none`, `openai`, `anthropic`.
- `ApiCalls` returns the response body even on HTTP errors (curl has `CURLOPT_FAILONERROR` commented out; `file_get_contents` uses `ignore_errors => true`). **It does not currently expose the HTTP status code to callers.**
- `ControllerApiKixote::promptGeneric()` passes `temperature`, `timeout`, and `reasoningEffort` to the adapter; these must be handled consistently by the new adapter.

### Proposed solution refinements

#### 1. New `OpenAiResponsesAdapter` class

- Endpoint: `POST {baseUrl}/responses`.
- Request body:
  ```json
  {
    "model": "gpt-5.4-mini",
    "input": [
      { "role": "system", "content": "..." },
      { "role": "user", "content": "..." }
    ],
    "temperature": 0.7,
    "max_output_tokens": 4000,
    "stream": false
  }
  ```
  - Use the array form for `input` (system + user messages) to match the existing chat-completions semantics used by Kixote.
  - Include `"stream": false` explicitly, consistent with the recent reasoning-models change.
  - Pass `temperature` only when the controller provides a non-null value.
  - Rename the token limit field to `max_output_tokens`.
- Response extraction:
  - Read `output` array.
  - Find the first item with `type: "message"`.
  - Return `content[0].text` after trimming.
  - Return a clear error if no message output is found.
- `listModels()`: reuse the same logic as `OpenAiAdapter` (`GET {baseUrl}/models`).
- `reasoning_effort`: the OpenAI Responses API structures reasoning differently than chat completions (`reasoning.effort` vs top-level `reasoning_effort`). For the first iteration, ignore the `reasoningEffort` parameter in `OpenAiResponsesAdapter` to keep the change minimal, or document it as unsupported.

#### 2. Adapter factory update

Extend `AiAdapter::create()`:

```php
return match ($adapter) {
    'anthropic'          => new AnthropicAdapter($baseUrl, $model, $apikey),
    'openai-responses'   => new OpenAiResponsesAdapter($baseUrl, $model, $apikey),
    default              => new OpenAiAdapter($baseUrl, $model, $apikey),
};
```

#### 3. Settings form

In `/system/typemill/settings/system.yaml`, add `openai-responses` as an option under `ai_adapter`:

```yaml
ai_adapter:
  type: select
  label: "AI Adapter"
  description: "Select 'OpenAI-compatible' for OpenAI, Ollama, LM Studio, OpenRouter, and most other providers. Select 'OpenAI Responses API' for providers that route GPT 5.x and newer models through the /responses endpoint. Select 'Anthropic' for Claude."
  options:
    none: 'None (disabled)'
    openai: 'OpenAI-compatible (OpenAI, Ollama, LM Studio, OpenRouter ...)'
    openai-responses: 'OpenAI Responses API (OpenCode Zen, OpenAI responses endpoint ...)'
    anthropic: 'Anthropic (Claude)'
```

The base URL, model, and API key fields remain unchanged.

#### 4. Better error messages for the existing chat adapter

Improve `OpenAiAdapter::chat()` error handling:

- When `data['error']` is present, read both `error.message` and `error.type`.
- If the error message/type indicates an internal/server error or is empty/ambiguous, append a hint:
  > "Provider returned an internal error. If you are using a model such as gpt-5.x on OpenCode Zen, try the OpenAI Responses API adapter instead."
- Wrap new user-facing strings with `Translations::translate()` so they can be extracted for localization.

### Important implementation considerations

1. **HTTP status codes are not currently available to adapters.**
   `ApiCalls` returns the body on HTTP 500, so the adapter can only inspect the JSON error body. To implement the proposed HTTP-500-specific hint accurately, we should either:
   - Extend `ApiCalls` with a minimal `getLastHttpStatus()` / `getStatusCode()` accessor, or
   - Accept body-based heuristics and adjust the acceptance criteria.
   Extending `ApiCalls` is preferred because it is a model-layer helper and makes the error hints reliable.

2. **Translations.**
   New user-facing error strings should be wrapped in `Translations::translate('...')` so translation files can pick them up.

3. **Backward compatibility.**
   Keep `OpenAiAdapter` unchanged except for the improved error hint. `AnthropicAdapter` remains untouched.

4. **Testing surface.**
   - Test the new adapter with an OpenCode Zen `gpt-5.4-mini` model.
   - Test the connection test and model-list endpoints.
   - Regression-test existing OpenAI-compatible providers (Ollama, OpenRouter, etc.).

### Files to touch

- `/system/typemill/Models/AiAdapter.php` — add `OpenAiResponsesAdapter`, update factory, improve `OpenAiAdapter` error hint.
- `/system/typemill/settings/system.yaml` — add `openai-responses` option.
- `/system/typemill/Models/ApiCalls.php` — optionally add HTTP status accessor (if body-based heuristics are not sufficient).

### Acceptance criteria review

- [ ] New `openai-responses` adapter exists and implements `AiAdapterInterface`.
- [ ] Adapter factory creates the correct implementation based on the configured adapter.
- [ ] Settings form exposes the new adapter option.
- [ ] Kixote prompts work with `gpt-5.4-mini` on OpenCode Zen when `openai-responses` is selected.
- [ ] `listModels()` still works for the new adapter (`GET {baseUrl}/models`).
- [ ] Connection test (`testConnection`) works for the new adapter.
- [ ] Error messages explain why a chat-completions call failed and point the user to the responses adapter.
- [ ] No regression for existing `openai` and `anthropic` adapters.
- [ ] No changes outside `/system/typemill/Models/AiAdapter.php`, `/system/typemill/settings/system.yaml`, and optionally `/system/typemill/Models/ApiCalls.php` unless absolutely necessary.

### Open questions / confirmations needed

1. Should we extend `ApiCalls` to expose HTTP status codes, or rely on JSON error-body heuristics for the endpoint-mismatch hint?
2. Should `reasoning_effort` be mapped/ignored for the Responses API in this first iteration?
3. Should new user-facing error strings be wrapped with `Translations::translate()` for future localization?
4. Should the settings form label explicitly mention OpenCode Zen, or stay generic ("OpenAI Responses API")?
5. Should `OpenAiResponsesAdapter` support `temperature: null` (omit the field) when the admin leaves temperature empty, matching the Anthropic behavior?

**Awaiting confirmation before implementation.**

1. Expose http status codes in ApiCalls only if there are no side effects and if it does not break any other functionality.
2. If reasoning_effort is complicated, then ignore it for now, if it is easy, then add it directly because user expect it to work if he edits it in the interface.
3. Yes, wrap with translations
4. stay generic
5. yes.

---

## Implementation Notes

### Confirmed decisions

1. Expose HTTP status codes in `ApiCalls` with no side effects.
2. Add `reasoning_effort` mapping for `OpenAiResponsesAdapter` because it is straightforward (`reasoning.effort`).
3. Wrap new user-facing strings with `Translations::translate()`.
4. Keep the settings form label generic ("OpenAI Responses API").
5. Support `temperature: null` (omit the field) in `OpenAiResponsesAdapter`, matching `AnthropicAdapter` behavior.

### Changes applied

#### `/system/typemill/Models/ApiCalls.php`

- Added private `?int $lastHttpStatus` property.
- Added public `getLastHttpStatus(): ?int` accessor.
- In `makeCurlCall()`: capture HTTP status via `curl_getinfo($curl, CURLINFO_HTTP_CODE)` after `curl_exec()`.
- In `makeFileGetContentsCall()`: parse and store status from `$http_response_header[0]` whenever available.
- No change to return values or existing error behavior.

#### `/system/typemill/Models/AiAdapter.php`

- Added `use Typemill\Static\Translations;`.
- Added `OpenAiResponsesAdapter` implementing `AiAdapterInterface`:
  - Endpoint: `POST {baseUrl}/responses`.
  - Request body: `{ model, input: [...], max_output_tokens, stream: false }` plus optional `temperature` and `reasoning.effort`.
  - Response extraction: iterates `output` for first `type: "message"` item and returns `content[0].text`.
  - `listModels()`: `GET {baseUrl}/models` with same logic as `OpenAiAdapter`.
- Updated `AiAdapter::create()` factory:
  - `'openai-responses' => new OpenAiResponsesAdapter(...)`
- Improved `OpenAiAdapter::chat()` error hints:
  - Detects internal/server errors from HTTP status >= 500, error message, or error type.
  - Appends a hint suggesting the Responses API adapter for GPT 5.x / OpenCode Zen scenarios.
  - Also appends hint when content is empty and HTTP status >= 500.
  - All new user-facing strings wrapped with `Translations::translate()`.

#### `/system/typemill/settings/system.yaml`

- Added `openai-responses: 'OpenAI Responses API'` option to `ai_adapter`.
- Updated description to mention the `/responses` endpoint generically.

### Verification

- `php -l` passed for `ApiCalls.php` and `AiAdapter.php`.
- Symfony YAML parser confirmed `system.yaml` is valid.
- Smoke tests against a local mock server passed:
  - Factory creates correct adapter for `openai`, `openai-responses`, `anthropic`.
  - `OpenAiResponsesAdapter::chat()` parses Responses API output correctly.
  - `OpenAiResponsesAdapter::listModels()` works.
  - `OpenAiAdapter` still works for chat-completions success.
  - `OpenAiAdapter` error for a gpt-5-style model includes the Responses API adapter hint.

### Result

Typemill now supports the OpenAI Responses API via a new adapter option. Users can select "OpenAI Responses API" in AI settings to use providers (e.g. OpenCode Zen) that route GPT 5.x models through `/responses`. The existing chat-completions adapter remains backward compatible and now surfaces a helpful hint when it detects an internal/server error that may indicate an endpoint mismatch.

---

## Test Results Anthrophic

Configured:

* AI-Adapter: Anthrophic (Claude)
* Api-Base-Url: https://api.anthropic.com/v1 
* Model: claude-sonnet-4-6
* Temperature: NULL

=> Result: works

* Changed selected model in Kixote info to claude-sonnet-5

=> Result: works, but unclear if it really used the overwrite model or still claude-sonnet-4-6

* Changed default model in settings to claude-sonnet-5
* Info shows now default model claude-sonnet-5 and selected model claude-sonnet-5

=> Result: error message "Anthropic did not return a valid answer."

* Changed selected model in info to claude-sonnet-4-6 with default model claude-sonnet-5

=> Result: error message "Anthropic did not return a valid answer."


So there are two problems: The selected model in the info does not seem to work and the model claude-sonnet-5 still returns an error.

---

## Follow-up Fixes

### Root causes identified

1. **Anthropic content parsing failed on newer models.**
   `AnthropicAdapter::chat()` assumed the first content block was always a text block (`$data['content'][0]['text']`). Newer Claude responses can return other block types first (e.g. `thinking`), causing the adapter to report *"Anthropic did not return a valid answer."* even when a valid text answer exists.

2. **Kixote model override was UI-only.**
   The Info tab stored the selected model in `localStorage` (`kixote_model_override`) and displayed it as an override, but it was **never sent to the backend**. Both the Generate prompt request (`/api/v1/prompt`) and the Help request (`/api/v1/kixote/help`) only used the configured default model.

### Changes applied

#### `/system/typemill/Models/AiAdapter.php`

- Updated `AnthropicAdapter::chat()` to iterate through the `content` array and return the first block with `type: "text"`.
- Wrapped the error message with `Translations::translate()`.

#### `/system/typemill/Controllers/ControllerApiKixote.php`

- Added private helper `applyModelOverride(?string $override): void` that validates the override (non-empty string, max 100 chars) before replacing `$this->aimodel`.
- Applied the override in `prompt()` from `$params['model']`.
- Applied the override in `help()` from `$body['model']`.

#### `/system/typemill/author/js/vue-kixote.js`

- In `submitPrompt()` (Generate tab), read `kixote_model_override` from `localStorage` and send it as `model` in the `/api/v1/prompt` payload.
- In `submitCommand()` (Help tab), send the same override as `model` in the `/api/v1/kixote/help` payload.

### Verification

- `php -l` passed for `AiAdapter.php` and `ControllerApiKixote.php`.
- `node --check` passed for `vue-kixote.js`.
- Smoke tests against the local mock server passed, including:
  - `AnthropicAdapter` correctly extracts text from a response where a `thinking` block precedes the `text` block.
  - Factory routing for all three adapters still works.
  - OpenAI Responses API adapter parsing still works.

### Notes

- These fixes were discovered while regression-testing the Anthropic adapter after the OpenAI Responses API changes, but they are independent issues that also affect production use of newer Anthropic models.
- Live verification with the actual `claude-sonnet-5` model is still recommended, but the parsing and override logic now matches the documented Anthropic Messages API response format.