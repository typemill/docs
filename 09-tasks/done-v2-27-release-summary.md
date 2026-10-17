# Typemill v2.27.0 Release Summary

Release v2.27 focuses on AI adapter compatibility, a more robust translation feature, Docker readiness, and three security fixes.

## OpenAI Responses API adapter

A new "OpenAI Responses API" option (`openai-responses`) was added to the AI settings, enabling providers like OpenCode Zen that route GPT 5.x models through the `/responses` endpoint instead of `/chat/completions`. `ApiCalls` now exposes the last HTTP status code so adapters can detect server errors, and the existing chat-completions adapter returns a helpful hint pointing to the new adapter when a model is likely served on the responses endpoint. Follow-up fixes from regression testing are included as well: the Anthropic adapter now returns the first text block instead of failing when a thinking block precedes it, and the model selected in the Kixote info tab is now actually sent to the backend for generate and help requests.

## Optional AI temperature setting

The temperature setting is now optional: when the field is left empty, the adapters omit the `temperature` parameter from the request entirely so the provider default is used. This solves the HTTP 400 errors returned by newer models such as Claude Sonnet 5, which reject non-default temperature values, without requiring a per-model mapping table. A stored value is still sent and clamped to 0.0–1.0, and the setting description explains that an empty value lets the provider decide.

## Kixote editor fixes for long articles

Two causes of cut-off articles were fixed: a CSS fix removes the vertical padding from the Kixote article textarea so autosize.js measures the correct height in Tailwind's border-box layout (the last lines were visually clipped), and the maximum AI output token budget was raised from 12,000 to 128,000, with new options from 16,000 up to 128,000 available in the system settings.

## Proxy detection for Docker

For the Docker image (prepared as a mittwald mStudio container template), proxy detection is now enabled by default via the `TYPEMILL_PROXY_DETECTION=true` environment variable, which is evaluated at runtime in the Settings model after the user settings are merged. This makes Typemill generate `https://` URLs for assets when running behind the Caddy reverse proxy instead of `http://` mixed-content URLs. The variable can be overridden at runtime, `settings.yaml` is never modified, and proxy detection remains opt-in for normal installations.

## Security: case-variant media download bypass

A follow-up to CVE-2026-71518 (reported by baeseungwon1010): on case-insensitive or Unicode-normalizing filesystems, requesting a case or encoding variant of a restricted file (e.g. `/media/files/Secret.pdf` instead of `secret.pdf`) bypassed the role restriction while still serving the same protected file to unauthenticated users. The download controller now derives the restriction lookup key from the `realpath()`-resolved path, so the access decision and the served file always use the same identity, closing case, Unicode, and encoding bypasses in one place; directory and symlink requests are also rejected explicitly.

## Security: ownership and publish checks on the article API

Two authorization gaps in the public article API were fixed: any user with `mycontent:read` could read the full draft body of arbitrary pages (including other users' unpublished drafts), and any authenticated user could retrieve metadata and navigation items for unpublished pages that return 404 on the public frontend. A new `publicApiIsAllowed()` helper aligns the API with the public frontend: unpublished pages return 404 for everyone except global content readers and the page owner, restricted published pages still respect `alloweduser`/`allowedrole`, and the `draft=true` branch now requires ownership when the caller lacks `content:read`.

## Security: SVG sanitizer hardening

A stored XSS in the SVG sanitizer (reported by Fernando Viana / capivara-research) was fixed: the sanitizer checked elements and attributes but not processing instructions, so an `<?xml-stylesheet?>` XSLT payload survived sanitization and executed JavaScript in the Typemill origin when an administrator opened the direct SVG link. The sanitizer now removes all processing instructions and comments, rejects SVGs containing a DOCTYPE declaration with a meaningful error message, and `loadSVG()` properly returns success or failure (parsing with `LIBXML_NONET` and suppressed libxml warnings); the duplicated sanitization code in `Media.php` was consolidated into a single helper. Already-stored SVGs are not cleaned retroactively, so re-upload or delete suspicious media after updating.

## Robust translation feature (multilang)

The languages/translation feature received a comprehensive robustness pass: all lang-tab buttons now lock while a request is in flight (with per-action labels), load and action errors are displayed instead of failing silently, and the translation index is invalidated whenever relevant project settings change or pages are created. Failed lookups self-heal by rebuilding the index once and retrying, phantom entries are cleaned up on unlink, and the create path no longer writes index entries when copying a page fails. Additionally, the multilang write endpoints are now guarded with `content create`/`content update` ACL rights and Valitron validation, homepages are included in the translation index and linked automatically (enabling hreflang tags and language switches on homepages), AI meta translation tolerates code-fenced answers and retries once automatically, and a stale extended-navigation cache that broke frontend language links after publishing was fixed.

## Setup creates missing folders

The setup controller no longer only checks but directly creates missing folders (settings, settings/users, content, data, cache, tmp, and all media folders) during the setup routine. This fixes Docker-based installations where mounted volumes do not contain required subfolders such as `settings/users`, which previously blocked the initial setup.
