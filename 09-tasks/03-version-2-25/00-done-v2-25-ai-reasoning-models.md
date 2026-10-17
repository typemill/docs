Good day, Mr. Schürmanns,

We are using the AI feature of Typemill with the following configuration:

- Typemill: 2.24.1
- AI Adapter: OpenAI-compatible
- Provider: Ollama 0.21.2
- Endpoint: `/v1/chat/completions`
- Model: `gemma4:31b`
- Streaming: disabled
- API key: none

The connection to Ollama is working in principle. Model requests and chat completions return HTTP 200, and Ollama processes the requests successfully. However, when using reasoning models, Typemill users sometimes receive no visible response at all or, after 120 seconds, only a generic communication error message.

The model returns both `message.content` and `message.reasoning`. If the token limit is reached during the reasoning phase, a valid response may look like this:

```json
{
  "choices": [
    {
      "finish_reason": "length",
      "message": {
        "content": "",
        "reasoning": "..."
      }
    }
  ]
}
```

Based on our analysis, the current Typemill adapter evaluates only `choices[0].message.content`. In addition, the request does not include any setting to control the model's reasoning behavior.

Ollama's OpenAI-compatible API officially supports the following parameter for this purpose:

```json
{
  "reasoning_effort": "none"
}
```

We manually tested the same request with this parameter:

- HTTP status: 200
- Response time: 0.45 seconds
- `finish_reason`: `stop`
- `message.content`: `OK`
- No reasoning content returned

Would it be possible to extend the OpenAI-compatible adapter with a reasoning configuration?

Our proposal:

- Add a **Reasoning** option with the levels **Off / Low / Medium / High**
- When **Off** is selected, include the parameter `"reasoning_effort": "none"` in the request
- Explicitly set `"stream": false` for non-streaming requests
- Make the request timeout configurable
- If `finish_reason` is `"length"` and `message.content` is empty, display a meaningful error message
- Do not expose the model's internal reasoning as the user-facing response

A working example request would be:

```json
{
  "model": "gemma4:31b",
  "messages": [
    {
      "role": "user",
      "content": "Reply only with OK."
    }
  ],
  "stream": false,
  "max_tokens": 128,
  "reasoning_effort": "none"
}
```

Official Ollama documentation:

https://docs.ollama.com/api/openai-compatibility

If needed, we would be happy to provide anonymized sample responses and additional response-time measurements.

---

## Review & Implementation Plan

### Current State Assessment

- The OpenAI-compatible adapter in `/system/typemill/Models/AiAdapter.php` currently only reads `choices[0].message.content`.
- It does not send `stream: false` explicitly (providers default to non-streaming, but being explicit is safer).
- It does not send `reasoning_effort`, so reasoning/thinking models default to reasoning.
- There is no special handling for `finish_reason === 'length'` when `message.content` is empty.
- The request timeout is already configurable via the `aitimeout` setting introduced in `done-v2-25-ai-timeout.md`. That part of the request is already implemented.

### Proposed Solution

#### 1. Add a new `ai_reasoning_effort` setting to the AI settings form

File: `/system/typemill/settings/system.yaml`

Add a `select` field under `fieldsetai` (e.g., after `aitimeout`):

```yaml
ai_reasoning_effort:
  type: select
  label: 'Reasoning effort'
  description: "For thinking/reasoning models only (e.g. Ollama, OpenAI o1/o3). 'Default' lets the provider decide. 'Off' disables reasoning. Other levels control reasoning depth."
  class: "w-half"
  options:
    '': 'Default (provider decides)'
    'none': 'Off'
    'low': 'Low'
    'medium': 'Medium'
    'high': 'High'
```

**Note:** Ollama's OpenAI-compatible endpoint supports `reasoning_effort` values `"high"`, `"medium"`, `"low"`, `"max"`, and `"none"`. The UI should provide Off/Low/Medium/High as requested, with an optional `''` (Default) value so existing installations stay backward compatible and only send the parameter when the admin explicitly chooses a level.

#### 2. Read the setting in the controller and pass it to the adapter

File: `/system/typemill/Controllers/ControllerApiKixote.php`

Add a private helper:

```php
private function getReasoningEffort(): ?string
{
    $value   = $this->settings['ai_reasoning_effort'] ?? '';
    $allowed = ['none', 'low', 'medium', 'high'];

    return in_array($value, $allowed, true) ? $value : null;
}
```

In `promptGeneric()`:

```php
$reasoningEffort = $this->getReasoningEffort();
$adapter = AiAdapter::create(...);
$answer  = $adapter->chat($systemMessage, $userMessage, $maxTokens, $temperature, $timeout, $reasoningEffort);
```

#### 3. Extend the adapter interface and implementations

File: `/system/typemill/Models/AiAdapter.php`

Update the interface signature:

```php
public function chat(
    string  $systemMessage,
    string  $userMessage,
    int     $maxTokens,
    float   $temperature,
    ?int    $timeout = null,
    ?string $reasoningEffort = null
): string|false;
```

In `OpenAiAdapter::chat()`:

- Add `'stream' => false` to `$postdata`.
- If `$reasoningEffort` is set, add `'reasoning_effort' => $reasoningEffort` to `$postdata`.
- After decoding the response:
  - If `finish_reason === 'length'` and `message.content` is empty, return a specific error: "The AI response was cut off because the maximum output length was reached. Please increase the output token limit or shorten the prompt."
  - Otherwise keep the existing generic error.
- Continue returning only `message.content`; never expose `message.reasoning` as the answer.

In `AnthropicAdapter::chat()`:

- Accept the new parameter in the signature for interface compatibility.
- Do not add `reasoning_effort` to the request body (this feature is scoped to the OpenAI-compatible adapter).
- Leave current behavior unchanged.

#### 4. Timeout is already configurable

No changes required. `ControllerApiKixote::getTimeout()` already reads `aitimeout` and passes it to the adapter.

### Files to Touch

- `/system/typemill/settings/system.yaml` — add `ai_reasoning_effort` form field.
- `/system/typemill/Controllers/ControllerApiKixote.php` — read setting and pass it to the adapter.
- `/system/typemill/Models/AiAdapter.php` — accept and send `reasoning_effort`, explicitly set `stream: false`, improve the `length`/`empty content` error message.

### Backward Compatibility

- Default value is empty/`null`, so `reasoning_effort` is only sent when the admin explicitly selects a level.
- Existing installations continue to behave exactly as before.
- The Anthropic adapter remains unaffected.

### Open Questions / Confirmations Needed

1. Should the UI include a **Default** option (recommended for backward compatibility) or strictly **Off / Low / Medium / High** as requested?
2. Should the `length`/`empty content` error message be translatable? The existing adapter errors are hard-coded English; should we keep that consistency or add translation support?
3. Should the reasoning setting also be applied to the connection test and model-list endpoints? The connection test uses `promptGeneric()`, so it will inherit the setting automatically. Model list does not need it.

**Awaiting confirmation before implementation.**

---

## Implementation Notes

Confirmed decisions:

1. UI includes a **Default** option for backward compatibility.
2. The `length`/`empty content` error message is hard-coded in English, matching the existing adapter error style.
3. Reasoning setting is **not** applied to the connection test or model list.

### Changes Applied

#### `/system/typemill/settings/system.yaml`

Added the `ai_reasoning_effort` select field under the AI fieldset with options:

- `''` — Default (provider decides)
- `none` — Off
- `low` — Low
- `medium` — Medium
- `high` — High

#### `/system/typemill/Models/AiAdapter.php`

- Updated `AiAdapterInterface::chat()` to accept `?string $reasoningEffort = null`.
- Updated `OpenAiAdapter::chat()`:
  - Added `'stream' => false` to the request body.
  - Added `reasoning_effort` only when a non-empty value is provided.
  - Added specific handling for `finish_reason === 'length'` with empty content, returning a clear error message.
  - Continues returning only `message.content`; `message.reasoning` is never exposed as the answer.
- Updated `AnthropicAdapter::chat()` to accept the new parameter for interface compatibility while leaving behavior unchanged.

#### `/system/typemill/Controllers/ControllerApiKixote.php`

- Added `getReasoningEffort(): ?string` helper that reads `ai_reasoning_effort`, validates it against `['none', 'low', 'medium', 'high']`, and returns `null` for the default/empty value.
- Updated `promptGeneric()` to retrieve the reasoning effort and pass it as the sixth argument to the adapter's `chat()` method.

### Verification

- `php -l` passed for both modified PHP files.
- Symfony YAML parser confirmed `system.yaml` is valid.

### Result

The OpenAI-compatible adapter now supports configurable reasoning effort, explicitly disables streaming, and provides a meaningful error when the response is cut off due to the output token limit. The Anthropic adapter remains unaffected, and existing installations keep their current behavior unless the admin explicitly changes the new setting.