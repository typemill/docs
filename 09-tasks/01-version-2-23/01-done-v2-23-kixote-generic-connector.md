2# Kixote AI Connector Refactoring Plan

## Goal

Refactor the current Kixote AI integration into a generic, provider-agnostic connector architecture that:

- supports privacy-friendly local AI systems like Ollama and LM Studio
- supports cloud AI providers like OpenAI and Claude
- removes provider-specific configuration and model maintenance
- simplifies the codebase significantly
- allows future AI providers without code changes
- positions Typemill as a local-first and privacy-focused AI-ready CMS

The final system should support two adapter types only:

1. OpenAI-Compatible Adapter
2. Anthropic-Compatible Adapter

All provider-specific branding and model logic should be removed from Typemill.

---

# Architecture Overview

## Current Problems

Current implementation issues:

- provider-specific logic everywhere
- duplicated prompt logic
- hardcoded model lists
- ongoing maintenance for model updates
- settings tightly coupled to brands
- difficult extension for local AI tools
- dead code (`promptKixote()`, `getKixoteJWT()`) that must be removed

---

## Target Architecture

Typemill should no longer know anything about:

- ChatGPT
- Claude
- Ollama
- LM Studio
- Mistral
- OpenRouter
- etc.

Instead, Typemill only knows:

- adapter type
- base URL
- model
- API key (optional for local providers)

---

# Adapter Types

## 1. OpenAI-Compatible Adapter

Supports:

- OpenAI
- Ollama
- LM Studio
- LocalAI
- vLLM
- OpenRouter
- most future providers

Endpoint format:

```text
POST {ai_base_url}/chat/completions
```

Required headers:

```text
Content-Type: application/json
Authorization: Bearer {ai_api_key}   (omitted if key is empty)
```

---

## 2. Anthropic-Compatible Adapter

Supports:

- Claude / Anthropic

Endpoint format:

```text
POST {ai_base_url}/messages
```

Required headers:

```text
Content-Type: application/json
x-api-key: {ai_api_key}
anthropic-version: 2023-06-01
```

Note: The `anthropic-version` header is mandatory for all Claude API calls and must be hardcoded in the adapter.

---

# New Configuration Structure

## Remove Old Settings

Remove from `system/typemill/settings/system.yaml` (`fieldsetai`):

```yaml
aiservice        # radio: none / chatgpt / claude
chatgptModel     # select with hardcoded model list
chatgptKey       # password
claudeModel      # select with hardcoded model list
claudeKey        # password
```

---

## Add Generic Settings

Replace with:

```yaml
ai_adapter       # select: openai / anthropic
ai_base_url      # text input
ai_model         # text input
ai_api_key       # password (optional for local providers)
```

### Exact YAML field definitions for `system/typemill/settings/system.yaml`

Replace the entire `fieldsetai` block with:

```yaml
fieldsetai:
  type: fieldset
  legend: AI
  fields:
    aiprivacy:
      type: paragraph
      label: "Notice"
      description: "Connect Typemill to any OpenAI-compatible AI provider (OpenAI, Ollama, LM Studio, OpenRouter, ...) or to Anthropic (Claude). For local providers like Ollama no API key is needed. If you use a cloud provider, user inputs will be sent to that external service. Ensure compliance with your local data privacy regulations. Each user will be asked for confirmation before using AI features. Read more in the [documentation](https://docs.typemill.net/admin-guide/ai-tab)"
    ai_adapter:
      type: select
      label: "AI Adapter"
      description: "Select 'OpenAI-compatible' for OpenAI, Ollama, LM Studio, OpenRouter, and most other providers. Select 'Anthropic' for Claude."
      options:
        none: 'None (disabled)'
        openai: 'OpenAI-compatible (OpenAI, Ollama, LM Studio, OpenRouter ...)'
        anthropic: 'Anthropic (Claude)'
    ai_base_url:
      type: text
      label: "API Base URL"
      placeholder: "http://localhost:11434/v1"
      description: "The base URL of your AI provider. Examples: https://api.openai.com/v1 (OpenAI), http://localhost:11434/v1 (Ollama), http://localhost:1234/v1 (LM Studio), https://api.anthropic.com/v1 (Claude). Do not add a trailing slash. Only point this to trusted endpoints."
      maxlength: 200
    ai_model:
      type: text
      label: "Model"
      placeholder: "llama3"
      description: "The model name as expected by your provider. Examples: gpt-4.1, llama3, mistral, claude-sonnet-4."
      maxlength: 100
    ai_api_key:
      type: password
      label: "API Key"
      autocomplete: new-password
      generator: false
      description: "Your API key. Leave empty for local providers like Ollama or LM Studio that do not require authentication. For security reasons your key is stored separately and will not be visible again after you leave this page."
    ai_provider_name:
      type: text
      label: "Provider Name"
      placeholder: "OpenAI"
      description: "A human-readable name for your AI provider (e.g. 'OpenAI', 'My Ollama Server'). This name is shown to users on the agreement screen. Leave empty to show the adapter type instead."
      maxlength: 100
    ai_provider_terms:
      type: text
      label: "Terms & Conditions URL"
      placeholder: "https://openai.com/policies/terms-of-use"
      description: "Optional link to the terms and conditions of your AI provider. Shown to users on the agreement screen. Leave empty for local providers that have no terms."
      maxlength: 500
    aitemperature:
      type: select
      label: 'Temperature (strict/creative)'
      description: "Default: 0.7 (0.1 = very strict, 1.0 = very creative)"
      class: "w-half"
      options:
        '0.1': '0.1'
        '0.2': '0.2'
        '0.3': '0.3'
        '0.4': '0.4'
        '0.5': '0.5'
        '0.6': '0.6'
        '0.7': '0.7'
        '0.8': '0.8'
        '0.9': '0.9'
        '1.0': '1.0'
    aioutputtoken:
      type: select
      label: 'Maximum output tokens'
      description: "Default: 4000 (output tokens determine the length/cost of answers)"
      class: "w-half"
      options:
        '2000': '2000 (Chat)'
        '4000': '4000 (Docs)'
        '6000': '6000 (Analysis)'
        '8000': '8000 (Research)'
        '10000': '10000 (High)'
        '12000': '12000 (Max)'
```

---

## Supported Configurations

### OpenAI Cloud

```json
{
  "ai_adapter": "openai",
  "ai_base_url": "https://api.openai.com/v1",
  "ai_model": "gpt-4.1",
  "ai_api_key": "sk-..."
}
```

### Ollama (local, no key required)

```json
{
  "ai_adapter": "openai",
  "ai_base_url": "http://localhost:11434/v1",
  "ai_model": "llama3",
  "ai_api_key": ""
}
```

### LM Studio (local, no key required)

```json
{
  "ai_adapter": "openai",
  "ai_base_url": "http://localhost:1234/v1",
  "ai_model": "mistral",
  "ai_api_key": ""
}
```

### Claude

```json
{
  "ai_adapter": "anthropic",
  "ai_base_url": "https://api.anthropic.com/v1",
  "ai_model": "claude-sonnet-4",
  "ai_api_key": "sk-ant-..."
}
```

---

# Implementation Steps

---

# Phase 1 — Update Settings Schema and Migration

## Goal

Replace all provider-specific settings fields in `system/typemill/settings/system.yaml` with the generic fields defined above.

---

## Tasks

### 1.1 Update `system/typemill/settings/system.yaml`

Replace the full `fieldsetai` block with the new field definitions shown above.

---

### 1.2 Migration Strategy: Read-Fallback in `setAiInfo()`

Typemill stores user-modified settings in `/settings/system.yaml` and secrets in `/settings/secrets.yaml`. There is no migration runner. Existing installations will have old keys (`aiservice`, `chatgptKey`, `claudeKey`, etc.) in those files after the schema update.

The migration must be handled transparently in `setAiInfo()` using a read-fallback:

```php
private function setAiInfo(): bool
{
    // Read new generic settings first
    $adapter = $this->settings['ai_adapter'] ?? null;
    $baseUrl  = $this->settings['ai_base_url'] ?? null;
    $model    = $this->settings['ai_model'] ?? null;

    $settingsModel = new Settings();
    $apikey = $settingsModel->getSecret('ai_api_key');

    // Fallback: migrate from old provider-specific settings
    if (!$adapter || $adapter === 'none') {
        $oldService = $this->settings['aiservice'] ?? null;
        if ($oldService === 'chatgpt') {
            $adapter = 'openai';
            $baseUrl  = $baseUrl ?: 'https://api.openai.com/v1';
            $model    = $model ?: ($this->settings['chatgptModel'] ?? null);
            $apikey   = $apikey ?: $settingsModel->getSecret('chatgptKey');
        } elseif ($oldService === 'claude') {
            $adapter = 'anthropic';
            $baseUrl  = $baseUrl ?: 'https://api.anthropic.com/v1';
            $model    = $model ?: ($this->settings['claudeModel'] ?? null);
            $apikey   = $apikey ?: $settingsModel->getSecret('claudeKey');
        }
    }

    $this->aiadapter      = $adapter;
    $this->aibaseurl      = rtrim($baseUrl ?? '', '/');  // normalize trailing slash
    $this->aimodel        = $model;
    $this->apikey         = $apikey;
    $this->aiprovider     = $this->settings['ai_provider_name'] ?? '';
    $this->aiproviderterms = $this->settings['ai_provider_terms'] ?? '';

    // adapter and base URL are required; model is required; api key is optional
    $missing = [];
    if (!$this->aiadapter || $this->aiadapter === 'none') { $missing[] = 'AI adapter'; }
    if (!$this->aibaseurl)                                 { $missing[] = 'API base URL'; }
    if (!$this->aimodel)                                   { $missing[] = 'AI model'; }

    if (!empty($missing)) {
        $this->error = 'Missing configuration: ' . implode(', ', $missing);
        return false;
    }

    return true;
}
```

Key points:
- `ai_api_key` is **not** required (local providers have no key)
- Trailing slashes on `ai_base_url` are normalized with `rtrim`
- Old secrets (`chatgptKey`, `claudeKey`) are read as fallback so existing installs keep working without any manual action
- `ai_provider_name` and `ai_provider_terms` are optional and default to empty string

Add two new private properties to the controller alongside the existing ones:

```php
private string $aiprovider      = '';
private string $aiproviderterms = '';
```

---

### 1.3 User Agreement — Per-Provider Model

Each user's YAML file stores which AI providers they have agreed to in an `aiservices` array. The stored value is the **agreement identifier** — either the human-readable provider name from `ai_provider_name` (e.g. `"OpenAI"`, `"My Ollama Server"`) when that field is non-empty, or the adapter type (`"openai"`, `"anthropic"`) as a fallback when no name is configured.

This design is intentional: an admin might run two different OpenAI-compatible providers (e.g. OpenAI and OpenRouter) each with different terms. Binding agreement to the adapter type alone would wrongly conflate them.

**Agreement identifier helper** — add to controller, used by both `getTokenStats()` and `agreeToAiService()`:

```php
private function getAgreementIdentifier(): string
{
    return !empty($this->aiprovider) ? $this->aiprovider : $this->aiadapter;
}
```

**`agreeToAiService()` stores the identifier:**

```php
$identifier = $this->getAgreementIdentifier();
// add $identifier to user's aiservices array if not already present
```

**`getTokenStats()` checks the identifier:**

```php
$identifier    = $this->getAgreementIdentifier();
$agreements    = $userdata['aiservices'] ?? [];
$useragreement = in_array($identifier, $agreements);
```

**Migration from old values:** Existing users with `"chatgpt"` or `"claude"` in their `aiservices` array will be asked to agree once after the upgrade. This is correct: the admin may have changed to a different provider, so re-agreement is appropriate. No `normalizeAgreements()` helper is needed.

---

# Phase 2 — Create AiAdapter Model

## Goal

Centralize all AI communication in a single model class, following the existing Typemill pattern where models handle external API communication (as `ApiCalls` already does).

---

## File Location

```text
/system/typemill/Models/AiAdapter.php
```

No new directory, no new namespace segment, no autoloader changes. The file sits alongside `ApiCalls.php` and all other models and is covered by the existing `Typemill\Models\` autoloading.

---

## 2.1 `AiAdapter.php` — Full Contents

The interface, both adapter classes, and the factory are all in one file. They are small and tightly coupled — splitting them across files would add navigation overhead with no practical benefit.

```php
<?php

namespace Typemill\Models;

/**
 * Contract for all AI provider adapters.
 */
interface AiAdapterInterface
{
    /**
     * Send a chat request and return the plain-text answer.
     *
     * @param string $systemMessage  The system/role instruction.
     * @param string $userMessage    The assembled user message (prompt + tagged content).
     * @param int    $maxTokens      Maximum output tokens.
     * @param float  $temperature    Temperature (0.0–1.0).
     * @return string|false          The answer string, or false on failure.
     */
    public function chat(
        string $systemMessage,
        string $userMessage,
        int    $maxTokens,
        float  $temperature
    ): string|false;

    public function getError(): string;
}

/**
 * Adapter for OpenAI-compatible APIs.
 * Supports: OpenAI, Ollama, LM Studio, LocalAI, vLLM, OpenRouter, and others.
 * Endpoint: POST {baseUrl}/chat/completions
 */
class OpenAiAdapter implements AiAdapterInterface
{
    private string $baseUrl;
    private string $model;
    private string $apikey;
    private string $error = '';

    public function __construct(string $baseUrl, string $model, string $apikey)
    {
        $this->baseUrl = rtrim($baseUrl, '/');
        $this->model   = $model;
        $this->apikey  = $apikey;
    }

    public function chat(string $systemMessage, string $userMessage, int $maxTokens, float $temperature): string|false
    {
        $url = $this->baseUrl . '/chat/completions';

        $headers = [];
        if (!empty($this->apikey)) {
            // API key is optional — local providers like Ollama do not require one
            $headers[] = "Authorization: Bearer {$this->apikey}";
        }

        $postdata = [
            'model'       => $this->model,
            'messages'    => [
                ['role' => 'system', 'content' => $systemMessage],
                ['role' => 'user',   'content' => $userMessage],
            ],
            'temperature' => $temperature,
            'max_tokens'  => $maxTokens,
        ];

        $api = new ApiCalls();
        $api->setTimeout(120);
        $response = $api->makePostCall($url, $postdata, $headers);

        if (!$response) {
            $this->error = 'Failed to communicate with AI provider: ' . $api->getError();
            return false;
        }

        $data = json_decode($response, true);

        if (isset($data['error'])) {
            $this->error = $data['error']['message'] ?? 'AI provider returned an error.';
            return false;
        }

        if (empty($data['choices'][0]['message']['content'])) {
            $this->error = 'AI provider did not return a valid answer.';
            return false;
        }

        return trim($data['choices'][0]['message']['content']);
    }

    public function getError(): string { return $this->error; }
}

/**
 * Adapter for the Anthropic API (Claude).
 * Endpoint: POST {baseUrl}/messages
 * Requires the anthropic-version header on every request.
 */
class AnthropicAdapter implements AiAdapterInterface
{
    private string $baseUrl;
    private string $model;
    private string $apikey;
    private string $error = '';

    public function __construct(string $baseUrl, string $model, string $apikey)
    {
        $this->baseUrl = rtrim($baseUrl, '/');
        $this->model   = $model;
        $this->apikey  = $apikey;
    }

    public function chat(string $systemMessage, string $userMessage, int $maxTokens, float $temperature): string|false
    {
        $url = $this->baseUrl . '/messages';

        $headers = [
            "x-api-key: {$this->apikey}",
            "anthropic-version: 2023-06-01",  // required by Anthropic on every request
        ];

        $postdata = [
            'model'       => $this->model,
            'system'      => $systemMessage,  // system prompt is a top-level field, not inside messages
            'messages'    => [
                ['role' => 'user', 'content' => $userMessage],
            ],
            'temperature' => $temperature,
            'max_tokens'  => $maxTokens,
        ];

        $api = new ApiCalls();
        $api->setTimeout(120);
        $response = $api->makePostCall($url, $postdata, $headers);

        if (!$response) {
            $this->error = 'Failed to communicate with Anthropic: ' . $api->getError();
            return false;
        }

        $data = json_decode($response, true);

        if (isset($data['error'])) {
            $this->error = $data['error']['message'] ?? 'Anthropic returned an error.';
            return false;
        }

        if (empty($data['content'][0]['text'])) {
            $this->error = 'Anthropic did not return a valid answer.';
            return false;
        }

        return trim($data['content'][0]['text']);
    }

    public function getError(): string { return $this->error; }
}

/**
 * Factory: instantiate the correct adapter from configuration.
 */
class AiAdapter
{
    public static function create(string $adapter, string $baseUrl, string $model, string $apikey): AiAdapterInterface
    {
        return match ($adapter) {
            'anthropic' => new AnthropicAdapter($baseUrl, $model, $apikey),
            default     => new OpenAiAdapter($baseUrl, $model, $apikey),
        };
    }
}
```

---

## 2.2 Usage in the Controller

```php
use Typemill\Models\AiAdapter;

// in promptGeneric():
$adapter = AiAdapter::create(
    $this->aiadapter,
    $this->aibaseurl,
    $this->aimodel,
    $this->apikey ?? ''
);

$answer = $adapter->chat($systemMessage, $userMessage, $maxTokens, $temperature);

if ($answer === false) {
    $this->error = $adapter->getError();
}
```

---

# Phase 3 — Replace Prompt Logic in Controller

## Goal

Remove all duplicated provider-specific methods and route everything through a single generic prompt method.

---

## 3.1 Remove

Delete:

```php
promptChatGPT()
promptClaude()
```

---

## 3.2 Responsibility Split: Content Assembly vs. Dispatch

The current `promptChatGPT()` and `promptClaude()` do two things:

1. **Assemble** the user message from `$prompt`, `$article`, an optional `$addition`, and an optional `$tag`
2. **Dispatch** the assembled message to the provider API

These two responsibilities must be separated. The caller is responsible for assembling the full `$userMessage` string. `promptGeneric()` only dispatches.

This is necessary because the translation feature uses **two distinct content blocks** in the same message:

- `autotrans()` article call: `<article>{markdown}</article>` only
- `autotrans()` meta call: `<article>{yaml}</article>` only
- `autotransUpdate()` article call: `<article>{original}</article>` + `<translation>{existing translation}</translation>`
- `autotransUpdate()` meta call: `<article>{original yaml}</article>` + `<translation>{existing yaml}</translation>`

A generic assembly helper would not be flexible enough to cover all these shapes without becoming as complex as the original. Keeping assembly in the caller is cleaner and more explicit.

---

## 3.3 Add `promptGeneric()`

```php
private function promptGeneric(
    string  $userMessage,
    ?string $systemMessageOverride = null
): string|false
{
    $systemMessage = $systemMessageOverride ?? $this->getSystemMessage();
    $maxTokens     = $this->getOutputBudget($userMessage);
    $temperature   = $this->getTemperature();

    $adapter = AiAdapter::create(
        $this->aiadapter,
        $this->aibaseurl,
        $this->aimodel,
        $this->apikey ?? ''
    );

    $answer = $adapter->chat($systemMessage, $userMessage, $maxTokens, $temperature);

    if ($answer === false) {
        $this->error = $adapter->getError();
    }

    return $answer;
}
```

`$systemMessageOverride` is required for the translation calls, which use a YAML-specific system message different from the default content-editing system message.

---

## 3.4 How Each Caller Assembles Its Message

Each call site builds the `$userMessage` before calling `promptGeneric()`. The pattern from the existing methods is preserved:

```php
// simple: one content block
$userMessage = $prompt . "\n<article>" . $article . "</article>";

// with an addition (e.g. example content or existing translation):
$userMessage = $prompt . "\n<article>" . $article . "</article>"
             . "\n<" . $tag . ">" . $addition . "</" . $tag . ">";
```

### Concrete call patterns

| Caller | userMessage structure |
|---|---|
| `prompt()` | `$prompt + <article>$article</article> [+ <example>$example</example>]` |
| `autotrans()` article | `$prompt + <article>$markdown</article>` |
| `autotrans()` meta | `$prompt + <article>$yaml</article>` + YAML system override |
| `autotransUpdate()` article | `$prompt + <article>$origMarkdown</article> + <translation>$transMarkdown</translation>` |
| `autotransUpdate()` meta | `$prompt + <article>$origYaml</article> + <translation>$transYaml</translation>` + YAML system override |

---

## 3.5 Call Flow

```text
Controller action (autotrans, prompt, etc.)
    ↓
assemble $userMessage (tagged XML structure, caller's responsibility)
    ↓
promptGeneric($userMessage, $optionalSystemOverride)
    ↓
AiAdapter::create(adapter, baseUrl, model, apikey)
    ↓
OpenAiAdapter::chat()  OR  AnthropicAdapter::chat()
    ↓
Provider API
    ↓
plain text answer returned to caller
```

---

## 3.6 Replace All `switch ($this->aiservice)` Blocks

There are five switch blocks in the controller across two methods:

1. `prompt()` — main prompt action
2. `autotrans()` — article translation
3. `autotrans()` — YAML/metadata translation (second AI call, YAML system message override)
4. `autotransUpdate()` — article update translation
5. `autotransUpdate()` — YAML/metadata update translation (second AI call, YAML system message override)

All five switch blocks are replaced with content assembly + a call to `promptGeneric()`. The two YAML translation calls pass a `$systemMessageOverride`. No other changes to the logic of any of these methods are required.

---

# Phase 4 — Simplify Token Handling

## Goal

Remove hardcoded per-model token limits while retaining a practical safety guard against oversized requests.

---

## 4.1 Delete

Remove completely:

```php
private array $aimodels
getTokenInfo()
getInputBudget()
getMaxBudget()
```

---

## 4.2 Retain `getOutputBudget()`

Keep the output budget as a simple configurable cap:

```php
private function getOutputBudget(string $content): int
{
    $hardCap = (int) ($this->settings['aioutputtoken'] ?? 4000);
    $hardCap = max(256, min(12000, $hardCap));
    return $hardCap;
}
```

**Trade-off note:** The original `getOutputBudget()` dynamically reduced the cap based on estimated input token count to stay within a model's context window. The simplified version passes the configured cap regardless of input size. This means very long inputs may produce an API error from providers with small context windows. This is an acceptable trade-off: providers return a clear error message, local models are typically used with long-context variants, and the old model-matching heuristic was fragile and required constant maintenance.

---

## 4.3 Retain Input Truncation for Example Content

The `prompt()` action truncates the optional `$example` content when the total input is too long. Without `getInputBudget()`, use a fixed conservative character limit:

```php
$maxInputChars = 60000; // ~15k tokens, safe for most providers
$allContent    = $prompt . $article . $example;
if (mb_strlen($allContent, 'UTF-8') > $maxInputChars)
{
    $overLimit = mb_strlen($allContent, 'UTF-8') - $maxInputChars;
    $keep      = mb_strlen($example, 'UTF-8') - $overLimit;
    $example   = $keep > 0 ? mb_substr($example, 0, $keep) : false;
}
```

This keeps the safety behaviour without per-model tables.

---

# Phase 5 — Update Frontend (Vue + Admin UI)

## Goal

Remove all hardcoded provider references from the Vue frontend.

---

## 5.1 User Agreement Screen

Remove the hardcoded `v-if="aiservice == 'chatgpt'"` / `v-if="aiservice == 'claude'"` ToS link blocks and the hardcoded `value="chatgpt"` attribute on the checkbox input (it is currently wrong — the `@change` handler passes `aiservice` directly, so the `value` attribute is unused).

The backend passes `providername` and `providerterms` inside the `tokenstats` object from `getTokenStats()` (see 5.2). The agreement screen renders the provider name dynamically and shows a terms link only when the URL is non-empty:

```html
<h2 class="text-xl font-bold mb-2">Agree to AI Usage</h2>
<label class="flex items-start mb-2 mt-2">
  <input
    type    = "checkbox"
    class   = "w-6 h-6 border-stone-300 bg-stone-200"
    @change = "agreeTo(aiservice)"
  >
  <span class="ml-2 text-sm">
    By enabling AI features, you agree to the terms and conditions of
    {{ tokenstats.providername || aiservice }}.
    Your prompts and article content will be sent to that provider.
    You can disable AI at any time in your user account.
  </span>
</label>
<div v-if="tokenstats.providerterms" class="text-sm mt-4">
  <p class="font-bold mt-2 mb-2">Terms:</p>
  <a :href="tokenstats.providerterms" class="text-teal-600" target="_blank">
    {{ tokenstats.providername || aiservice }} Terms &amp; Conditions
  </a>
</div>
```

When neither `ai_provider_name` nor `ai_provider_terms` is configured (typical for local providers), the screen shows only the adapter type and no links — which is correct.

---

## 5.2 Token Stats / Usage Tab

The `getTokenStats()` backend currently returns provider-specific billing dashboard URLs. Replace with a neutral structure that includes the provider name and terms URL so the agreement screen (5.1) and usage tab can both use them:

```json
{
  "aiservice":     "openai",
  "useragreement": true,
  "tokenstats": {
    "adapter":       "openai",
    "model":         "llama3",
    "providername":  "My Ollama Server",
    "providerterms": ""
  }
}
```

`providername` is the value of `ai_provider_name` (empty string when not configured). `providerterms` is the value of `ai_provider_terms` (empty string when not configured).

The `tab-usage` Vue template currently reads `tokenstats.service` and `tokenstats.url` — both keys no longer exist. Replace the template body with:

```html
<div v-if="aiservice">
  <p class="py-2">Active adapter: <strong>{{ tokenstats.adapter }}</strong></p>
  <p class="py-2">Model: <strong>{{ tokenstats.model }}</strong></p>
  <p v-if="tokenstats.providername" class="py-2">
    Provider: <strong>{{ tokenstats.providername }}</strong>
  </p>
  <p class="py-2 text-sm text-stone-500">
    Usage statistics are available in your provider's dashboard.
  </p>
</div>
<div v-else>
  <p class="py-2">No AI service has been activated.</p>
  <p class="py-2">You can enable and configure one in the system settings to start using AI features.</p>
</div>
```

Billing URL links are removed entirely. For cloud providers, users manage billing in their own provider dashboard.

---

## 5.3 No-AI-Service Message

The current message in `tab-generate` says:

> Select an AI service. Choose a model. Enter your API key.

Update to match the new field names:

> Select an AI adapter, enter the base URL and model name, and optionally enter an API key in the System Settings → AI tab.

---

# Phase 6 — Remove Dead Code

## Goal

Clean up unused and broken code.

---

## Delete

Remove entirely from `ControllerApiKixote.php`:

```php
promptKixote()    // broken stub, references undefined $settings and $licensedata
getKixoteJWT()    // only called by promptKixote(), not used anywhere else
```

---

# Phase 7 — Add Connection Test Endpoint

## Goal

Allow admins to verify their AI configuration from the settings UI without having to run a real prompt.

---

## 7.1 Add Route

In `system/typemill/routes/api.php`, add:

```php
$group->post('/kixote/test', ControllerApiKixote::class . ':testConnection')
      ->setName('api.kixote.test')
      ->add(new ApiAuthorization($acl, 'user', 'update')); # admin only
```

ACL level `user/update` is appropriate because only admins who can change system settings should be able to trigger this.

---

## 7.2 Implement `testConnection()`

```php
public function testConnection(Request $request, Response $response): Response
{
    $aisettings = $this->setAiInfo();
    if (!$aisettings) {
        $response->getBody()->write(json_encode(['message' => $this->error]));
        return $response->withHeader('Content-Type', 'application/json')->withStatus(400);
    }

    $answer = $this->promptGeneric('Reply with the single word: ok');

    if ($answer === false) {
        $response->getBody()->write(json_encode([
            'message' => 'Connection failed: ' . $this->error
        ]));
        return $response->withHeader('Content-Type', 'application/json')->withStatus(400);
    }

    $response->getBody()->write(json_encode([
        'message' => 'Connection successful',
        'model'   => $this->aimodel,
        'adapter' => $this->aiadapter,
    ]));
    return $response->withHeader('Content-Type', 'application/json')->withStatus(200);
}
```

---

# Implementation Checklist

## Backend

- [ ] Update `system/typemill/settings/system.yaml` — replace `fieldsetai` block (includes new `ai_provider_name` and `ai_provider_terms` fields)
- [ ] Add `$aiprovider` and `$aiproviderterms` private properties to `ControllerApiKixote`
- [ ] Create `system/typemill/Models/AiAdapter.php` — interface, both adapters, factory in one file
- [ ] Refactor `setAiInfo()` — new fields + migration fallback + trailing slash normalization + read `ai_provider_name` and `ai_provider_terms`
- [ ] Add `getAgreementIdentifier()` helper — returns `$this->aiprovider` if non-empty, else `$this->aiadapter`
- [ ] Update `getTokenStats()` — use `getAgreementIdentifier()` for agreement check; return `tokenstats` with `adapter`, `model`, `providername`, `providerterms`; remove provider-specific switch block and billing URLs
- [ ] Update `agreeToAiService()` — store `getAgreementIdentifier()` value in user's `aiservices` array
- [ ] Add `promptGeneric()` with optional system message override
- [ ] Replace all 5 `switch ($this->aiservice)` blocks with `promptGeneric()` calls
- [ ] Replace `getOutputBudget()` with simplified version
- [ ] Replace `getInputBudget()` usage in `prompt()` with fixed character limit
- [ ] Delete `promptChatGPT()`, `promptClaude()`
- [ ] Delete `promptKixote()`, `getKixoteJWT()`
- [ ] Delete `$aimodels`, `getTokenInfo()`, `getInputBudget()`, `getMaxBudget()`
- [ ] Add `testConnection()` action
- [ ] Add route for `POST /kixote/test` in `api.php`

## Frontend (vue-kixote.js)

- [ ] Remove `v-if="aiservice == 'chatgpt'"` / `v-if="aiservice == 'claude'"` ToS link blocks
- [ ] Remove hardcoded `value="chatgpt"` from agreement checkbox input
- [ ] Replace agreement text with dynamic `tokenstats.providername || aiservice` and conditional `tokenstats.providerterms` link
- [ ] Update "no AI service" instruction text to match new field names (adapter, base URL, model, optional API key)
- [ ] Update `tab-usage` template to use `tokenstats.adapter`, `tokenstats.model`, `tokenstats.providername` — remove `tokenstats.service` and `tokenstats.url` references

---

# Expected Result

## Codebase

- no duplicated provider logic
- no hardcoded model lists to maintain
- clean adapter boundary
- easier extension for future providers

## Product

Typemill becomes a privacy-friendly AI-ready documentation system that works with any OpenAI-compatible or Anthropic-compatible AI provider, including fully local setups.

## Supported AI Systems (out of the box, without code changes)

- OpenAI
- Claude / Anthropic
- Ollama
- LM Studio
- LocalAI
- vLLM
- OpenRouter
- any future OpenAI-compatible provider

---

# Implementation Report

> Status: **complete and tested**

## Files changed

| File | Change |
|---|---|
| `system/typemill/settings/system.yaml` | Replaced entire `fieldsetai` block |
| `system/typemill/Models/AiAdapter.php` | New file — interface, adapters, factory |
| `system/typemill/Controllers/ControllerApiKixote.php` | Full rewrite |
| `system/typemill/routes/api.php` | Two new routes added |
| `system/typemill/author/js/vue-kixote.js` | Agreement screen, no-AI message, tab-info component |

---

## Phase 1.1 — Settings YAML

Replaced the entire `fieldsetai` block. Removed: `aiservice`, `chatgptModel`, `chatgptKey`, `claudeModel`, `claudeKey`. Added: `ai_adapter` (select), `ai_base_url` (text), `ai_model` (text), `ai_api_key` (password), `ai_provider_name` (text, optional), `ai_provider_terms` (url text, optional). Temperature and output token fields unchanged.

## Phase 1.2 — `setAiInfo()` and new properties

Added private properties `$aiadapter`, `$aibaseurl`, `$aimodel`, `$apikey`, `$aiprovider`, `$aiproviderterms`. `setAiInfo()` reads new generic keys first and falls back to old `aiservice`/`chatgptKey`/`claudeKey` for existing installs. Trailing slash on `ai_base_url` is normalized with `rtrim`. API key is not required (local providers).

## Phase 1.3 — Per-provider user agreement

Added `getAgreementIdentifier()`: returns `$this->aiprovider` when non-empty, else `$this->aiadapter`. `agreeToAiService()` stores the identifier in the user's `aiservices` array. `getTokenStats()` checks the identifier. Old `"chatgpt"` / `"claude"` entries are stale after upgrade — users re-agree once, which is correct.

## Phase 2 — `AiAdapter.php`

New file at `system/typemill/Models/AiAdapter.php`. Contains:
- `AiAdapterInterface` — `chat()` and `listModels()` contract
- `OpenAiAdapter` — `POST {baseUrl}/chat/completions`, `GET {baseUrl}/models`; Authorization header omitted when API key is empty
- `AnthropicAdapter` — `POST {baseUrl}/messages`, `GET {baseUrl}/models`; `anthropic-version: 2023-06-01` header always sent
- `AiAdapter` factory — `create()` returns `OpenAiAdapter` by default, `AnthropicAdapter` for `'anthropic'`

## Phase 3 — `promptGeneric()` and switch block removal

Removed `promptChatGPT()`, `promptClaude()`. Added `promptGeneric(string $userMessage, ?string $systemMessageOverride)`. Each of the five former switch blocks (`prompt()`, `autotrans()` ×2, `autotransUpdate()` ×2) now assembles `$userMessage` locally and calls `promptGeneric()`. YAML translation calls pass `$systemMessageOverride`.

Also fixed a bug in the original code: `<article>` closing tags were written as `<article>` (opening) instead of `</article>` — corrected in both `autotrans()` and `autotransUpdate()`.

## Phase 4 — Token handling

Removed: `$aimodels` array, `getTokenInfo()`, `getInputBudget()`, `getMaxBudget()`. Simplified `getOutputBudget()` returns the configured cap (256–12 000) without per-model context window math. Input truncation in `prompt()` uses a fixed 60 000-character limit (~15k tokens).

## Phase 5 — Frontend

**Agreement screen:** Removed hardcoded `v-if="aiservice == 'chatgpt'"` / `v-if="aiservice == 'claude'"` link blocks and wrong `value="chatgpt"` on checkbox. Now shows `aiinfo.providername || aiservice` and a conditional terms link from `aiinfo.providerterms`.

**No-AI message (`tab-generate`):** Updated to list adapter, base URL, model, optional API key steps.

**`tab-info` component:** Was `tab-usage`. Renamed in the tab list and component definition. Replaced `tokenstats.service` / `tokenstats.url` references.

## Phase 6 — Dead code removed

Deleted `promptKixote()` and `getKixoteJWT()` (broken stubs with undefined variables).

## Phase 7 — Connection test

Added `testConnection()` controller action and `POST /kixote/test` route (admin-only, `user/update` ACL). Sends a minimal `'Reply with the single word: ok'` prompt and returns adapter + model on success.

---

## Post-plan additions

### `aiinfo` rename (was `tokenstats`)

The JSON key returned by `getTokenStats()` was renamed from `tokenstats` to `aiinfo` throughout: controller response, Vue root data property, prop bindings in all four components, and template expressions. The route URL `/api/v1/tokenstats` and PHP method name `getTokenStats()` are kept to avoid breaking the API contract.

**`aiinfo` shape:**

```json
{
  "adapter":       "openai",
  "model":         "llama3",
  "providername":  "My Ollama Server",
  "providerterms": ""
}
```

### Model list endpoint (`GET /kixote/models`)

**Why:** Both OpenAI-compatible and Anthropic APIs expose a standard `GET {baseUrl}/models` endpoint. Ollama, LM Studio, OpenRouter, OpenAI, and Anthropic all implement it. This allows users to browse and select from actually available models without hardcoded lists.

**Backend:**

- `AiAdapterInterface::listModels(): array|false` — added to interface and both adapter implementations
- `OpenAiAdapter::listModels()` — calls `GET {baseUrl}/models`, returns sorted array of `['id' => string]` from `data[]`
- `AnthropicAdapter::listModels()` — calls `GET {baseUrl}/models` with `x-api-key` and `anthropic-version` headers, returns array of `['id' => string]` from `data[]` in provider order (newest first)
- `ControllerApiKixote::getModels()` — new action, returns `{ "models": [ {"id": "..."}, ... ] }`
- Route: `GET /api/v1/kixote/models` → `getModels()`, ACL: `mycontent/update` (author level, same as prompt)

**Frontend (`tab-info`):**

- "Load models" / "Refresh" button triggers `GET /api/v1/kixote/models`
- Model list rendered as a button list; selected model highlighted in teal
- Selection stored in `localStorage` under key `kixote_model_override`
- On mount, localStorage value is restored; configured model is shown as the default
- "Reset to configured model" link clears the localStorage entry

**Note:** The localStorage model override is stored in the browser only — it is not sent to the backend automatically. A follow-up would wire the stored value into the `prompt()` request payload so the backend uses it. Currently the configured `ai_model` is always used server-side.

---

# Optional Future Enhancements

## Auto-Detect Adapter from URL

If `ai_base_url` contains `anthropic.com`, automatically suggest `anthropic` adapter in the UI.

## Streaming Support

- streaming responses
- live generation display
- typing indicators

## Embedding Support

```text
POST /v1/embeddings
```

Useful for semantic search, RAG, Ask-The-Docs.
