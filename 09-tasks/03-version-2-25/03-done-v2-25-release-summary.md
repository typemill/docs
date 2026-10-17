# Typemill v2.25.0 Release Summary

## Configurable AI request timeout

The AI assistant now has a configurable request timeout in the admin panel under **Settings → AI settings**. A new `aitimeout` setting replaces the previous hard-coded 120-second limit, allowing administrators to choose values between 120 and 600 seconds. Both the OpenAI-style and Anthropic-style adapters in `AiAdapter.php` accept the timeout parameter, so long-running requests (such as proofreading very long articles) no longer abort prematurely.

## Reasoning-effort control for OpenAI-compatible models

A new `ai_reasoning_effort` setting lets administrators control reasoning behavior for thinking/reasoning models via the OpenAI-compatible adapter. The adapter explicitly sets `stream: false` on non-streaming requests and only sends the `reasoning_effort` parameter when a level is selected (`none`, `low`, `medium`, or `high`). When a response is cut off due to the output token limit (`finish_reason: length` with empty `message.content`), the adapter now returns a clear, meaningful error message instead of a generic communication error.

## Export-safe static asset pipeline for plugins

Typemill introduces `onExportHtmlLoaded`, a dedicated event that enables plugins to replace dynamic, browser-dependent content (such as Mermaid, PlantUML, or math formulas) with static images before EPUB, PDF, or static-site exports. The `Plugin` base class provides two helpers—`generateStaticAsset()` and `getStaticAssetPath()`—that hash source content, cache generated assets under `/cache/generated/`, and return either public URLs or local filesystem paths. This keeps `/media` reserved for user uploads and makes `/cache` the disposable home for derived export assets.

## Plugin developer guide for static export assets

A new plugin-developer guide explains how to make dynamic content export-safe using the `onExportHtmlLoaded` event. The documentation covers both rendering pipelines, the two events (`onHtmlLoaded` for browsers, `onExportHtmlLoaded` for exports), the helper methods available in the base plugin class, and concrete examples for Mermaid, PlantUML, and math plugins. It also describes the recommended `/cache/generated/` storage location, content-hash naming, and common pitfalls when writing generator callables.

## Fix for 2FA brute-force protection bypass

A security issue in the authcode-based two-factor authentication flow has been fixed. Previously, the per-user attempt counter was reset to zero every time a new authcode was generated, which allowed an attacker with valid credentials to cycle login requests and gain unlimited guesses. The counter is now decoupled from the authcode lifecycle and a new `authcodelocked` timestamp enforces a 15-minute cooldown after five failed attempts. The lockout is cleared only after successful 2FA validation or after the cooldown expires.

## Improved validation for multi-project base settings

When multi-project mode is enabled but the **Base Project ID** or **Base Project Label** fields are left empty, the system no longer fails silently. The settings save controller now returns a validation error directly under the affected fields, preventing the settings from being saved until both fields are filled. This change keeps the user in control of project IDs and labels while making the configuration requirement obvious.

## Signal theme landingpage refactor

The `/themes/signal` theme has been refactored with a new configurable landingpage. The `signal.yaml` settings now include fieldsets for hero, news, customer stories, solutions, capabilities, comparison, ecosystem, pricing, and getting-started sections, plus position fields to control section order. The existing footer logic is preserved but styled to match the prototype, and all assets (CSS, JS, fonts) remain self-contained within the theme. No external resources are used.

## Simplified 2FA authcode input

The six separate digit input fields on the authcode login form have been replaced by a single input field named `authcode`. The field accepts exactly six digits (`0-9`), supports copy-paste, and is validated with a single `/^[0-9]{6}$/` rule. The corresponding controller logic and Twig template have been updated, making the 2FA login experience faster and less error-prone for users.
