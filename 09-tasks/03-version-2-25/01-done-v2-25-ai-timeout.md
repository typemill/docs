ich benutze claude KI unter anderem zur Überprüfung der Rechtschreibung, Zeichensetzung usw.. Bei meinen teils sehr langen Texten auf energytools.de benötigt Claude deutlich mehr als 120 sec zur Überprüfung, weshalb ich bis zur Version 2.22. in der ControllerApiKixote.php timeout von 120 auf 300 hochgesetzt habe. Die Anzahl der token habe ich auf 12000 gesetzt. Das funktionierte gut, meine längsten Texte brauchten etwa 220 bis 240 sec. bis zur Auslieferung. Seit der Version 2.23 geht das nicht mehr, weil ich keine Möglichkeit gefunden habe timeout hochzusetzten. Bei 120 sec ist Schluss, claude bricht ab ohne Überprüfung. Lässt sich das timeout in den neueren Versionen der ControllerApiKixote.php auf andere Weise hochsetzen?

---

## Plan: Make AI timeout configurable (and verify token length)

### Problem
Since version 2.23 the HTTP timeout for AI requests is hard-coded to **120 seconds** inside `AiAdapter.php` (lines 77 and 184). Users with very long articles (e.g. spell-checking via Claude) need **300 seconds or more**.

The output-token limit is already editable via the admin setting `aioutputtoken` (clamped to max 12000). The missing piece is the **timeout**.

### Proposed solution (editable)

1. **Add a new setting `aitimeout` to the system settings form**  
   File: `/system/typemill/settings/system.yaml`  
   Add a `select` (or `number`) field directly below `aioutputtoken` with sensible steps, e.g.:
   ```yaml
   aitimeout:
     type: select
     label: 'AI request timeout'
     description: "Maximum time in seconds to wait for an AI response."
     class: "w-half"
     options:
       '120': '120 sec (default)'
       '180': '180 sec'
       '240': '240 sec'
       '300': '300 sec'
       '360': '360 sec'
       '420': '420 sec'
       '480': '480 sec'
       '600': '600 sec'
   ```

2. **Expose the timeout from the controller**  
   File: `/system/typemill/Controllers/ControllerApiKixote.php`  
   - Add a private helper `getTimeout(): int` that reads `$this->settings['aitimeout']`, falls back to `120`, and clamps the value (e.g. min 30, max 600).
   - Pass the returned value into the `AiAdapter` when calling `chat()`.

3. **Accept a timeout parameter in `AiAdapter`**  
   File: `/system/typemill/Models/AiAdapter.php`  
   - Change the signature of `chat(...)` to accept an optional `?int $timeout = null` argument.
   - Replace the hard-coded `$api->setTimeout(120);` with `$api->setTimeout($timeout ?? 120);`.
   - Do this for **both** adapter drivers inside the file (OpenAI-style and Anthropic-style paths, lines ~77 and ~184).

4. **Verify `ApiCalls.php`**  
   It already supports `$api->setTimeout(int $timeout)`, so no changes are needed there.

### Minimal fallback (if editable is too much)
If adding a UI field is not desirable right now, the **minimal** change is:
- In `AiAdapter.php`, replace the two occurrences of `setTimeout(120)` with `setTimeout(300)`.
- This immediately solves the user's problem, but affects all installations.

### Token length note
The token limit is already configurable via the existing `aioutputtoken` setting (admin panel → AI settings). The controller already clamps it to a maximum of **12000**, which matches the user's requirement. No extra work is needed for tokens unless a higher clamp is desired.

### Files to touch
- `system/typemill/settings/system.yaml` (add form field)
- `system/typemill/Controllers/ControllerApiKixote.php` (read setting & pass it)
- `system/typemill/Models/AiAdapter.php` (accept timeout parameter)

---

## Implementation Description

The following changes were implemented to make the AI request timeout configurable from the admin panel:

### 1. New admin setting `aitimeout`
Added to `system/typemill/settings/system.yaml` directly below the existing `aioutputtoken` field:
- A `select` dropdown labeled **"AI request timeout"**
- Options: 120, 180, 240, 300, 360, 420, 480, 600 seconds
- Default: 120 sec (preserving backward compatibility)
- Description: "Default: 120 sec (increase for very long articles)"

### 2. Controller changes
In `system/typemill/Controllers/ControllerApiKixote.php`:
- Added a new private helper `getTimeout(): int` that reads `$this->settings['aitimeout']`, defaults to `120`, and clamps the value between `30` and `600` seconds.
- Updated `promptGeneric()` to retrieve the timeout via `getTimeout()` and pass it as the 5th argument to `$adapter->chat(...)`.

### 3. Adapter changes
In `system/typemill/Models/AiAdapter.php`:
- Updated the `AiAdapterInterface::chat()` signature to accept an optional `?int $timeout = null` parameter.
- Updated both `OpenAiAdapter::chat()` and `AnthropicAdapter::chat()` signatures accordingly.
- Replaced the hard-coded `$api->setTimeout(120);` calls with `$api->setTimeout($timeout ?? 120);` in both adapters.
- Model-list calls (`listModels()`) intentionally keep their shorter 30-second timeout unchanged.

### 4. Token length
No changes were required. The existing `aioutputtoken` setting already supports up to **12000** tokens (the controller clamps to `max(256, min(12000, $value))`), which satisfies the user's requirement.

### Result
Users can now set the AI timeout directly in the admin panel under **Settings → AI settings**, choosing values up to 600 seconds. This resolves the issue where Claude (or other providers) would abort long requests after the hard-coded 120-second limit.
