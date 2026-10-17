If no ai service is configured, then the user still sees:

* Checkboxes for ai consent in the user profiles.
* AI buttons at the top of each content page that opens kixote ai.
* Update and create buttons with ai-icon in the translation lang tab if multilanguage is enabled.

Users who do not want ai and did not configure an AI service in the ai tab should not see any of these ai features.

---

## Plan to Fix

### Goal
If `ai_adapter` is not set (or set to `none`), or if `ai_base_url` / `ai_model` are missing, the user must not see:
1. The AI-consent checkbox list in user profiles.
2. The magic-wand button in the blox/raw editor.
3. The AI-icon on create/update buttons in the translation language tab.

### 1. Add a central backend check: `Settings::isAiConfigured()`

`ControllerApiKixote::setAiInfo()` already validates the exact same rules, but it is private and lives in a controller. We need a reusable model method so other controllers can gate UI without duplicating logic.

- **File:** `system/typemill/Models/Settings.php`  
- **Action:** Add `public function isAiConfigured(): bool` that checks:
  - `ai_adapter` is set and not `'none'`
  - `ai_base_url` is non-empty
  - `ai_model` is non-empty
  - (include the existing fallback for legacy `aiservice = 'chatgpt'` / `'claude'` so old configurations still work)

### 2. Remove AI-consent fields from user forms

The user profile form is generated dynamically from `user.yaml` in `User::getUserFields()`. This is the perfect place to strip the field when AI is off.

- **File:** `system/typemill/Models/User.php`  
- **Action:** In `getUserFields()`, after loading `$userfields`, call `isAiConfigured()`. If it returns `false`, `unset($userfields['aiservices'])`.  
- **Result:** The checkbox disappears from the account page, user-edit page, and new-user API form automatically.

### 3. Hide the magic-wand button in blox and raw editors

The editor Vue apps (`vue-blox.js`, `vue-raw.js`) need a boolean injected from the backend.

- **File:** `system/typemill/Controllers/ControllerWebAuthor.php`  
- **Action:** In both `showBlox()` and `showRaw()`, add `'aiconfigured' => (new Settings())->isAiConfigured()` to the `jsdata` array.

- **Files:** `system/typemill/author/js/vue-blox.js`, `system/typemill/author/js/vue-raw.js`  
- **Action:**  
  - Add `aiconfigured: data.aiconfigured` to the `data()` object.  
  - Wrap the `<button @click.prevent="openkixoteai()">` with `v-if="aiconfigured"`.

### 4. Hide AI icons in the translation language tab

The `tab-lang` component in `vue-meta.js` already has `autotranslateActive()`, but it only checks `this.settings.autotranslate`. If that setting is enabled while no AI service is configured, the icons still appear.

- **File:** `system/typemill/author/js/vue-meta.js`  
- **Action:** Update `autotranslateActive()` to:
  ```js
  return this.settings.autotranslate && data.aiconfigured;
  ```
- **Result:**  
  - The **create** button loses its magic-wand icon (`v-if="autotranslateActive"`).  
  - The **update** button disappears entirely because `showUpdate()` already depends on `autotranslateActive`.

### 5. Files to modify

| File | Change |
|------|--------|
| `system/typemill/Models/Settings.php` | Add `isAiConfigured()` |
| `system/typemill/Models/User.php` | Unset `aiservices` when AI is off |
| `system/typemill/Controllers/ControllerWebAuthor.php` | Inject `aiconfigured` into `jsdata` |
| `system/typemill/author/js/vue-blox.js` | `v-if="aiconfigured"` on AI button |
| `system/typemill/author/js/vue-raw.js` | `v-if="aiconfigured"` on AI button |
| `system/typemill/author/js/vue-meta.js` | Check `data.aiconfigured` in `autotranslateActive()` |

### 6. Verification

- **No AI configured:** Open account page, editor, and translation tab → no AI UI elements visible.  
- **AI configured (new generic fields):** All UI elements visible.  
- **AI configured (legacy fields like `aiservice = chatgpt`):** Still visible because `isAiConfigured()` handles the migration fallback.  
- **Partial AI config (missing model or URL):** Treated as not configured, UI hidden.

This plan keeps changes minimal, respects the existing architecture (no global state, no core refactoring), and centralizes the configuration check in the model layer.