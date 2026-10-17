For some newer models like anthrophic sonnet-5 I get an error message that the temperature is deprecated. 

Any way to solve this reliably? For me it is unclear why they do not just ignore it, because If I choose another model, temperature might be valid and I do not want to map all services and models again. 

---

## Agent Review & Implementation Plan

### Problem Confirmation
Anthropic's newer models (Claude Sonnet 5, Claude Opus 4.7+) reject the `temperature` sampling parameter when it is set to a non-default value and return a HTTP 400 error. The Anthropic docs state that the default value or omitting the parameter is accepted, but e.g. `temperature: 0.7` fails. OpenAI-compatible providers and older Anthropic models still accept and expect `temperature`.

This confirms the reported behavior: the same Typemill configuration works for one model and breaks for another, so a global hard-coded model-to-parameter mapping is undesirable.

### Proposed Solution
Make the `temperature` parameter optional. When `aitemperature` is empty/null, the adapters omit the `temperature` key from the request payload entirely. Typemill already adds an empty option to select fields that stores `null`, so no form-definition change is required.

Why this approach:
- No extra checkbox or new setting needed.
- No changes to `system.yaml` form structure (the empty select option already exists).
- No per-model mapping table (as requested).
- Backward compatible for existing installations: a stored temperature value (e.g. `0.7`) is still sent.
- Works across adapters: both Anthropic and OpenAI-compatible adapters simply skip the key when the value is `null`.

### Files to Modify
1. `/system/typemill/Models/AiAdapter.php`
   - Change `AiAdapterInterface::chat()` signature: `float $temperature` → `?float $temperature`.
   - In `OpenAiAdapter::chat()` only add `temperature` to the POST payload when `$temperature !== null`.
   - In `AnthropicAdapter::chat()` only add `temperature` to the POST payload when `$temperature !== null`.

2. `/system/typemill/Controllers/ControllerApiKixote.php`
   - Change `getTemperature(): float` to `getTemperature(): ?float`.
   - Read `aitemperature` from settings.
   - If the stored value is empty/null/empty-string, return `null` so the adapters omit the parameter.
   - Otherwise cast to float and clamp to `0.0 – 1.0`.
   - Remove the `0.7` fallback so that "no value selected" unequivocally means "do not send temperature".

3. `/system/typemill/settings/system.yaml` (optional text-only update)
   - Not required for the fix, but the current description says "Default: 0.7". Since the new default becomes the provider's own default, update the description to something like:
     ```yaml
     description: "0.1 = very strict, 1.0 = very creative. Leave empty to let the provider decide (required for newer Anthropic models like Sonnet 5)."
     ```

### Downsides of Removing the Fallback
- **Behavior change for new installs/users who clear the field**: instead of Typemill's previous `0.7`, the provider's API default is used. For OpenAI and Anthropic the API default is `1.0`, which produces more creative/random output than `0.7`.
- **Existing users are unaffected**: anyone who already saved `0.7` (or any other value) will continue to send it.
- If the previous deterministic default is important, keep a fallback of `0.7` only when the setting key is completely missing (old installs). That, however, re-introduces the ambiguity you noted: an empty selection would then behave differently from a never-configured field.

Given that the goal is to let users omit temperature for Anthropic Sonnet 5+, removing the fallback is the cleanest and most predictable behavior.

### Alternative Considered
Automatic retry on a 400 error with a temperature-related message was considered. It would remove the need for any UI change, but it doubles the latency and API cost for every affected call and relies on fragile error-message parsing. Reusing the existing temperature select is simpler and keeps controllers thin.

### Testing Notes
- Verify that a selected temperature (e.g. `0.7`) still results in the `temperature` key being present in the request payload for both adapters.
- Verify that an empty temperature selection results in the `temperature` key being omitted from the payload.
- Verify the connection test and Kixote prompts work for an Anthropic Sonnet 5 model with temperature omitted.
- There is no dedicated test suite for `AiAdapter` currently, so manual verification via the admin AI connection test is the practical approach.

### Implementation Status
Confirmed by user and applied on 2026-09-10.

Changes made:
- `/system/typemill/Models/AiAdapter.php`
  - `AiAdapterInterface::chat()` now accepts `?float $temperature`.
  - `OpenAiAdapter::chat()` and `AnthropicAdapter::chat()` only include `temperature` in the payload when it is not `null`.
- `/system/typemill/Controllers/ControllerApiKixote.php`
  - `getTemperature()` now returns `?float`.
  - Empty/null values return `null`; no `0.7` fallback.
  - Non-empty values are still clamped to `0.0 – 1.0`.
- `/system/typemill/settings/system.yaml`
  - Updated `aitemperature` description to explain that an empty value lets the provider decide and is required for newer Anthropic models.

PHP syntax checks passed for both modified PHP files.
