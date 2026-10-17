Some problems: 

The project feature with the translation mode is activated. The ai-translation is activated. 

* The buttons in the lang tab of a page to translate or update a translation should be deactivated after click until a response is there.
* When I disable the project and translation feature, change some pages and structures, and enable the feature again, then the translation index is outdated and sometimes shows buttons and actions although I do not see the page or it even does not exists, only in the index. When I click the update button, then it does not find the page, but also does not return any error message and does not update the interface.
* There should be an option to delete and recreate the translation index manually in case the translation index is wrong.
* There should be a mechanism to regenerate the translation index in case there are errors or pages from the index cannot be found.

---

# Review & Implementation Plan

Status: **IMPLEMENTED - see section 8 for the implementation report.**
All findings below were verified against the source (file paths + line numbers refer to the state of the code BEFORE implementation).

## 0. Architectural note

The translation feature (project mode `languages` + AI translation) is implemented entirely in core (`/system/typemill`), not as a plugin. The translation index is core data (`/data/multilang/index.txt`, a serialized array keyed by base-page `pageid`). Since plugins must not modify core behavior and the index is core-owned, the fixes below necessarily touch core. They are classified as bug fixes / completions of the existing feature (missing invalidation, missing error handling, missing recovery path), not new features. The only change that adds a new UI surface is the "Rebuild translation index" button (item 3) - placement is flagged as an open question.

## 1. Root-cause analysis (verified)

### Problem 1: buttons not deactivated after click

Component `tab-lang` in `system/typemill/author/js/vue-meta.js:332-891`:

* The **update button (lines 479-490) has no `:disabled` binding at all**.
* `storeTranslation()` (789) sets **no busy state**; only `updateTranslation()` (826) and `autotranslate()` (845) set `this.translate = langKey`. The create-request and the chained autotranslate-request run without any lock.
* The "Translating ..." overlay (420-428) covers only the affected row; **all other rows' buttons stay clickable** (concurrent AI calls possible).
* After success/error of update/autotranslate, `formData` is never reloaded, so the UI does not reflect server state (contributes to Problem 2).

### Problem 2: outdated index, missing errors, no interface update

Invalidation gaps - `MultilangExtension` (`system/typemill/Extensions/MultilangExtension.php`) subscribes only `onPageDeleted`/`onPageRenamed`/`onPageSorted`:

* **Settings changes never invalidate the index.** `ControllerApiSystemSettings::updateSettings()` contains no multilang logic. Toggling `projects` off/on, changing `projectinstances` or `baseprojectid` leaves a stale index in place - exactly the reported scenario. The extension is only registered when `projects == 'languages'` (`system/typemill/system.php:234-237`), so all changes made while the feature is disabled go unnoticed.
* **`onPageCreated` is not subscribed** - pages created while the feature is active never enter the index until it happens to be deleted elsewhere. Consequence: `POST /api/v1/multilang` (`createMultilang`, `ControllerApiMultilang.php:185-202`) fails with 404 "We did not find the page id in the mulitlangindex" for every page created since the last index build.
* **Meta-tab renames are invisible**: `ControllerApiAuthorMeta::updateMeta()` renames post files on `manualdate` change (line 351) **without dispatching `onPageRenamed`** (the article controller does dispatch it, `ControllerApiAuthorArticle.php:989`) - URLs in the index become stale.

Failed lookups are not handled:

* `ControllerApiKixote::autotrans` (543-549) / `autotransUpdate` (687-703) return 404 "page not found" when an indexed URL no longer resolves, but **nothing rebuilds the index and the UI does not refresh**.
* `ControllerApiMultilang::deleteMultilang` (349-350) uses `$targetInfo['path']` **without a null check** - if the translation page no longer exists, `getSourceInfo()` returns `false` and the code silently operates on garbage (meta never rewritten, no error message).
* `createMultilang` **ignores the return value of `copyPage()`** (line 270; `copyPage` returns storage error strings at 475/490/509/536) and writes the index entry anyway - this *creates* phantom entries.
* `createMultilang` references an **undefined `$result`** in the base-folder error response (208-213) - the API returns `error: null` instead of the actual storage error.
* `checkBaseFolder()` (405-442) overwrites the `createFolder` result without checking it, and the comment at line 419 marks **unimplemented logic**: the copied language-root `index.yaml` keeps the base home's `pageid` and gets no `translation_for`, so language home pages can never be linked and duplicate pageids exist on disk.

Invisible errors in the UI:

* `loadTranslations()` (783-787) and `loadTranslationIndex()` (763-766) write `this.message` / `this.messageClass`, but **neither is declared in `data()` nor rendered in the template** - load errors are invisible.
* When `GET /api/v1/multilang` returns 200 with `multilangDefinitions: false` (language page not linked to a base page) or fails, `formDefinitions` stays `null`/`false` and the template crashes on `formDefinitions.fields` - **blank tab without any message**.
* The raw editor's jsdata lacks `project` (`ControllerWebAuthor::showRaw()` lines 181-191 vs `showBlox()` line 111) - `tab-lang` renders the wrong variant in the raw editor.

### Problem 3: no manual delete/recreate option

* The backend already exists: `DELETE /api/v1/clearmultilang` (`routes/api.php:100`, `ControllerApiGlobals::clearMultilang()` 118-129, ACL `system update` = manager) deletes the index file, and the index is **lazily recreated** by `GET /api/v1/multilang(index)` when the file is missing (`ControllerApiMultilang::getFreshMultilangIndex()` 377-403).
* But it is only reachable through the Kixote chat command "clear multilang" (`author/js/vue-kixote.js:80-99`) - undiscoverable, and the command description is truncated mid-sentence.

=> it might be enough in kixote chat command, because if we fix other errors, the manual recreation should not be often needed.

### Problem 4: no regeneration mechanism on errors

* A **stale-but-present** index is never refreshed: the lazy rebuild only triggers when the file does not exist at all.
* No code path verifies that indexed URLs still resolve; failed lookups just error out (or fail silently, see above).

## 2. Implementation plan

### Phase 1 - UI busy states & error surfacing (Problem 1 + error display of Problem 2)

File: `system/typemill/author/js/vue-meta.js` (component `tab-lang`)

1. Busy state for all actions: set `this.translate = langKey` at the start of `storeTranslation`, `updateTranslation`, `autotranslate` and `unlinkTranslation`; clear it in every `then`/`catch` (including the `storeTranslation` -> `autotranslate` chain, so the whole chain stays locked).
2. Add `isBusy()` helper (true while `this.translate !== false`). Add `:disabled="isBusy(langKey)"` to the update button and extend `canUseButtons()` / `canCreate()` with the busy check, so **all action buttons in the tab lock while any request is in flight** (prevents concurrent AI calls; per-row-only locking is the weaker alternative - see open questions).
3. Make errors visible: declare `message`/`messageClass` in `data()` and render an error box at the top of the tab (same pattern as other components). Guard the form with `v-if="formDefinitions && formDefinitions.fields"` and show a hint instead of a broken/blank tab when `multilangDefinitions` is false.
4. Refresh state after actions: after `updateTranslation`/`autotranslate` success **or** error, call `loadTranslations()` so buttons/URLs reflect server state (also picks up server-side index rebuilds).
5. Fix typo "tranlating ..." (846) and route user-facing strings through `$filters.translate()`.

### Phase 2 - complete index invalidation (root cause of Problem 2)

6. `MultilangExtension`: subscribe `onPageCreated` -> `deleteIndex()` (consistent with the existing delete-and-lazy-rebuild strategy).
7. `ControllerApiAuthorMeta::updateMeta()`: dispatch `onPageRenamed` after a successful `renamePost()` (same event data shape as `ControllerApiAuthorArticle.php:989`).
8. `ControllerApiSystemSettings::updateSettings()`: after a successful save, if `projects`, `projectinstances`, `baseprojectid` or `baseprojectlabel` changed compared to the previous settings, call `Multilang::deleteMultilangIndex()`. Fixes the reported disable -> change -> re-enable scenario and added/removed languages.

### Phase 3 - self-healing & robustness (Problem 4 + error paths of Problem 2)

9. Move the rebuild logic from the private `ControllerApiMultilang::getFreshMultilangIndex()` into a public method on the `Multilang` model (e.g. `createFreshIndex($settings, $urlinfo)`) so all controllers can use it (removes duplication, keeps controllers thin).
10. Self-heal on failed lookups (rebuild once, then retry):
    * `createMultilang` / `deleteMultilang`: pageid not found in index -> rebuild index -> retry; only if still missing, return a clear error.
    * `autotrans` / `autotransUpdate` (`ControllerApiKixote`): `getItemForUrl()` fails for an indexed URL -> rebuild index -> retry once; if it still fails, return 404 with a clear message **and the fresh multilangData** so the UI can refresh.
11. `deleteMultilang`: null-check `getSourceInfo()`; if the translation page no longer exists, still clear the index entry and respond with a success message ("translation page not found, index entry removed") - normal use then heals phantom entries.
12. `createMultilang`: check the return value of `copyPage()` - on error return 400 with the storage error and **do not** write the index entry; fix the undefined `$result` (use the actual error from `checkBaseFolder`).
13. `autotrans` / `autotransUpdate`: check return values of `saveDraftMarkdown()` and `updateMeta()`; return 500 with the storage error instead of a silent "Success" on partial failure.
14. `checkBaseFolder()`: check the `createFolder` result before it is overwritten; implement the marked-unimplemented rewrite of the copied language-root `index.yaml` (assign a fresh unique `pageid` via the existing `updateMeta()` helper, link with `translation_for`), so language home pages become linkable and no duplicate pageids exist.

### Phase 4 - manual delete/recreate (Problem 3)

Please skip phase 4 / 15 and do not add a button because we can already recreate it in the kixote interface with a command.

15. Add a **"Rebuild translation index"** button in the system settings (`author/js/vue-system.js`), following the existing slot pattern of the "send testmail" button (lines 20-26):
    * Shown only when `formData.projects == 'languages'` (attached to a field of the languages fieldset).
    * Click -> `DELETE /api/v1/clearmultilang` -> `GET /api/v1/multilangindex` (triggers + verifies the lazy rebuild) -> success/error message inline.
    * No new backend endpoint needed; manager-only by ACL, and the settings page is already the manager area.

## 3. Related defects found during review (small, recommend same pass)

* Overview table hardcodes `{{ row.en }}` (`vue-meta.js:564`) - breaks for any `baseprojectid` other than `en`; use the configured base project id.
* `ControllerWebAuthor::showRaw()` jsdata lacks `project` (line 181) - `tab-lang` broken in the raw editor; add it like in `showBlox()` (line 111).
* Cleanup: `Multilang::updateMultilangIndex()` (`Multilang.php:224`) is dead code; controllers pass an ignored argument to `getMultilangIndex()` (`ControllerApiMultilang.php:53/101`).

## 4. Out of scope / for separate decision (NOT planned without explicit confirmation)

* **ACL**: `POST`/`DELETE /api/v1/multilang` are authorized with `mycontent read` only (`routes/api.php:115-116`) although they create/copy pages and rewrite meta - inconsistent with article routes (`content create`/`delete`). Tightening could break existing setups.
* **Valitron**: multilang endpoints use manual truthiness checks instead of Valitron (architecture requires Valitron); client-supplied `path` is stored into the index without server-side format validation.
* Server-side check of the `autotranslate` setting / per-user AI agreement in the autotrans endpoints.
* `getEditorPath()` adds the language prefix to the base version (self-admitted in comment, `vue-meta.js:702`).
* Concurrency: two parallel GETs can lazily rebuild the index simultaneously (last write wins). Considered acceptable for a derived, deterministic cache - not planned to lock.

## 5. Verification plan (after implementation)

* `php -l` on every changed PHP file (repo has no test suite).
* Manual QA with languages mode (2+ languages) and AI translation configured:
  1. Lang tab: click create/update/unlink - all buttons must lock until the response arrives; no concurrent clicks possible; loading errors show as visible message.
  2. Create a new page while the feature is active - creating a translation for it must work (no "pageid not found").
  3. Change `manualdate` of a base page (meta tab) - index invalidated/rebuilt, lang tab URLs correct.
  4. Disable project feature, delete/rename pages and change structure, re-enable - lang tab and overview must not offer actions for non-existing pages; unlink on a phantom entry must succeed and remove it; update on a missing page must return a visible error and refresh the tab.
  5. Add/remove a language in settings - index recreated.
  6. "Rebuild translation index" button in system settings - index file recreated, overview consistent afterwards.
  7. Raw editor: lang tab renders the correct variant (`project` branch).

## 6. Open questions (decisions needed before implementation)

1. **Busy scope**: lock ALL action buttons in the lang tab while any translation request is in flight (recommended, prevents concurrent AI calls), or lock only the affected row?
2. **Rebuild button placement**: system settings (recommended - manager area, existing slot pattern), or in the lang tab "Translation overview" (would need the user's role exposed to the frontend, since the clear endpoint is manager-only)?
3. **Scope of Phase 3**: include the robustness fixes 13-14 (silent partial failures, unimplemented language-home linking) now, or limit this pass to the four reported problems (then 13-14 move to a follow-up task)?
4. Confirm the out-of-scope list (especially ACL tightening for the multilang write endpoints).

## 7. Answers to 6

1. Yes, lock all action buttons.
2. Do not add another button to delete the language index, we can use existing command in kixote
3. Yes, include 13 and 14
4. No, we should implement ACL tightening and valitron checks.

---

# 8. Implementation Report (2026-09-13)

All approved plan items are implemented. 11 files changed, no new files, no new routes/endpoints, no database, no build steps. `php -l` passes on all changed PHP files, `node --check` passes on both changed JS files.

## 8.1 What was changed

### Phase 1 - UI busy states & error surfacing (`system/typemill/author/js/vue-meta.js`, component `tab-lang`)

* All four actions (`storeTranslation`, `updateTranslation`, `autotranslate`, `unlinkTranslation`) now set a busy state (`this.translate = langKey`) before the request and clear it in every `then`/`catch`; the `storeTranslation` -> `autotranslate` chain stays locked for its entire duration.
* New `isBusy()` helper; the update button got `:disabled="isBusy()"` (it had none at all), and `canUseButtons()`/`canCreate()` include the busy check - **all action buttons in the tab lock while any request is in flight** (answer to open question 1).
* The busy overlay now shows a per-action label (`Creating ...` / `Translating ...` / `Updating ...` / `Unlinking ...`) via the new `busyLabel` data property; typo "tranlating ..." fixed.
* `message`/`messageClass` are now declared in `data()` and rendered as an error box at the top of the tab - load errors from `loadTranslations()`/`loadTranslationIndex()` are visible (previously written to undeclared properties, i.e. invisible).
* The form is guarded with `v-else-if="formDefinitions && formDefinitions.fields"` plus a fallback hint, and `refreshEditData()` returns early when definitions are missing - the template no longer crashes on `null.fields` when a language page is not linked to a base page or the API fails.
* After `updateTranslation`/`autotranslate` success **or** error, `loadTranslations()` runs (now returns its promise) and the success/error message is set afterwards, so buttons/URLs always reflect the server state (this also picks up server-side index rebuilds and heals stale rows).
* `unlinkTranslation` shows the server message (e.g. "Translation page not found, index entry removed").
* User-facing strings routed through `$filters.translate()`.

### Phase 2 - complete index invalidation

* `MultilangExtension` now subscribes `onPageCreated` -> `deleteIndex()` (`system/typemill/Extensions/MultilangExtension.php`); comment in `system/typemill/system.php` updated.
* `ControllerApiSystemSettings::updateSettings()` deletes the multilang index via `Multilang::deleteMultilangIndex()` when `projects`, `baseprojectid`, `baseprojectlabel` or `projectinstances` changed - fixes the reported disable -> change -> re-enable scenario and added/removed languages.
* **Plan item 7 (dispatch `onPageRenamed` in the meta controller) was dropped after re-verification**: `Meta::renamePost()` only changes the sort-order/date prefix of the filename, and Typemill URLs are built from the slug only (`Folder.php:221-229`) - the URL and therefore the index entry remain valid. Dispatching a rename event would trigger needless full index rebuilds on every `manualdate` change.

### Phase 3 - self-healing & robustness

* The rebuild logic moved from the private controller method into the model: `Multilang::createFreshIndex($settings, $urlinfo)` (`system/typemill/Models/Multilang.php`), a faithful port of the old `getFreshMultilangIndex()`; the dead `updateMultilangIndex()` was removed and the bogus argument in `getMultilangIndex()` calls cleaned up.
* Self-heal "rebuild once, then retry" in all four write/action paths:
  * `createMultilang` / `deleteMultilang`: missing index -> fresh build; pageid not found -> rebuild + retry -> only then a clear error.
  * `ControllerApiKixote::autotrans` / `autotransUpdate`: missing index -> fresh build; pageid not found -> rebuild + retry; `getItemForUrl()` failure for base or translation URL -> rebuild + retry with the fresh URL; final failures return 404 with a clear message **and** the fresh `multilangData` so the UI can refresh.
* `deleteMultilang`: `getSourceInfo()` result is null-checked; when the translation page no longer exists, the phantom index entry is still cleared and the response explains it ("Translation page not found, index entry removed") - normal use now heals phantom entries.
* `createMultilang`: `copyPage()` return value is checked - on storage error the request fails with 400 and **no index entry is written** (this was the phantom-entry generator); the undefined `$result` in the base-folder error response is fixed (now returns the actual storage error, status 400); a missing base page (`getSourceInfo()` false) returns a clear 404 instead of operating on `false`.
* `checkBaseFolder()`: `createFolder()` result is checked before being overwritten; the previously unimplemented rewrite of the copied language-root `index.yaml` is implemented - the language home gets a fresh unique `pageid` and is linked to the base home via `translation_for` (when the base home has a pageid), so language home pages are now linkable and no duplicate pageids are created.
* `copyPage()`: the folder branch checks `createFolder()` before copying into it.
* `autotrans` / `autotransUpdate`: return values of `saveDraftMarkdown()` and `Meta::updateMeta()` are checked (500 with the storage error instead of a silent "Success"); `Yaml::parse()` of the AI meta answer is wrapped in try/catch and an invalid AI response returns 400 instead of a PHP fatal or a silent skip.

### ACL tightening (`system/typemill/routes/api.php:115-116`)

* `POST /api/v1/multilang` (creates/copies pages): `mycontent read` -> **`content create`** (author and up, same as article creation).
* `DELETE /api/v1/multilang` (unlink, rewrites meta of existing pages): `mycontent read` -> **`content update`** (editor and up, same as article discard).
* Behavioral change to be aware of: contributors can no longer create/unlink translations, and authors can no longer unlink (they can still create and translate). GET endpoints stay `mycontent read`.

### Valitron validation (`system/typemill/Models/Validation.php`, wired into all endpoints)

* New validators: `multilangCreate` (pageid/lang/path), `multilangDelete` (pageid/lang/url), `multilangGet` (pageid/url), `autotransInput` (pageid/lang) - all following the existing codebase style; GET `/multilangindex` reuses `articleUrl`.
* All multilang and autotrans endpoints validate via these methods and return 400 with `message`/`errors` in the standard format; the manual truthiness checks and the misleading "Prompt is missing or invalid." message are gone.
* Server-side check that `lang` is a configured project instance (the former TODO comment in `createMultilang`).
* Client-supplied `path`/`url` are now format-validated (slug pattern) before being stored into the index.

### Related fixes (section 3 of the plan)

* Overview table: hardcoded `{{ row.en }}` -> `{{ row[settings.baseprojectid] }}`.
* `ControllerWebAuthor::showRaw()`: `project` added to jsdata (plus the `$project` assignment it needs), so `tab-lang` renders the correct variant in the raw editor.
* Kixote command description completed: "It will be recreated automatically with the next pageview in the author area."

## 8.2 Verification performed

* `php -l` on all 9 changed PHP files: clean. `node --check` on both changed JS files: clean.
* Runtime smoke test (CLI with real autoload): all new Valitron validators accept valid input and reject invalid input (bad path characters, missing pageid, too-short pageid); `Multilang::createFreshIndex()` built a correct 19-entry index from the real content folder (keys `en`, `fr`, `parent`), stored it, and `getMultilangIndex()` returned the identical structure (serialize roundtrip works). Test artifacts (`/data/multilang`, an auto-created `content/_fr/` from the project-navigation setup) were removed afterwards; `/data/navigation` unchanged.
* Manual QA checklist from section 5 still applies for full acceptance (needs a languages-mode installation with 2+ languages and AI configured); item 6 (rebuild button) is obsolete per the decision to skip Phase 4.

## 8.3 Remaining known limitations (unchanged, out of scope per decisions)

* No server-side check of the `autotranslate` setting / per-user AI agreement in the autotrans endpoints.
* `getEditorPath()` still adds the language prefix to the base version (comment in code).
* Two parallel requests can trigger the lazy rebuild simultaneously (last write wins) - accepted for a derived, deterministic cache.
* Kixote `help()` returns errors under an `error` key instead of `message` (inconsistent with all other endpoints).

## 8.4 Homepage translation (added 2026-09-13 after follow-up discussion)

**Problem found during testing:** home pages (base and language) were never part of the translation index, because the index is built from the draft navigation, which only contains folder *contents* - a folder's own `index` page (the home) never appears in it. Consequences: the lang tab on the home page demanded a meaningless slug (create always failed with "page id not found"), the AI-update button could not work, and hreflang/language-switch links were missing on homepages. The language home itself is auto-created as a stub (`# Welcome / Content`) by `Navigation::setProject()` as soon as any `/lang/...` URL is requested.

**Agreed design (minimal variant):** the language homepage is created and linked automatically; the lang tab on home shows no slug editing - just the visit link, the AI-update button and a hint.

Implementation:

* `Multilang::createFreshIndex()` (`system/typemill/Models/Multilang.php`):
  * New `addBaseHomeToIndex()`: adds the base home entry (base URL `/`) to the index; a missing home `pageid` is generated and persisted via `Meta::addMetaDefaults()` (which self-persists), so it survives rebuilds.
  * New `addLanguageHomeToIndex()`: adds the language home (`/de` etc.) and links it to the base home - if the language home has no meta yet (the auto-created stub), its `index.yaml` is initialized with a fresh `pageid` + `translation_for` = base home pageid; if it is already linked, the existing link is kept and nothing is written (rebuilds do not churn content in the steady state).
  * The language loop now skips the base project id if someone misconfigured it as a project instance (previously `setProject()` would have created a bogus `/_en/` folder).
  * Precondition verified: `getItemForUrl()` resolves `/` and `/de` to the home items directly (Navigation.php:405-410), so `autotrans`/`autotransUpdate` work on homes without further changes; `Meta::getMetaData`/`updateMeta` handle home items via `pathWithoutType`.
* `tab-lang` (`system/typemill/author/js/vue-meta.js`), on home rows:
  * Both input fields disabled (`isInputDisabled` includes `isHome()`).
  * Create button hidden (unlink was already hidden for home).
  * Visit link always active on home.
  * AI-update button appears automatically once the home is indexed (`showUpdate` = existing translation + autotranslate active).
  * Hint per row: "The homepage for this language has been created automatically."
* Bonus: `ControllerWebFrontend` reads the same index, so hreflang alternates and the theme language switch now also work on homepages (base and language), without further changes.

Verification (runtime smoke test with a throwaway `fr` project instance against the real content folder):

* Base home entry in index: `en='/'`, `fr='/fr'`, `parent=null` - ok.
* Language home meta auto-initialized: `content/_fr/index.yaml` with fresh `pageid` + `translation_for` = base home pageid - ok.
* Regular pages unaffected: 20 entries (19 pages + home) - ok.
* Rebuild idempotent: second build produced identical index keys and kept the language home pageid stable (no churn) - ok.
* Live files untouched (checksummed before/after): `content/index.yaml`, `content/_de/`, the live `/data/multilang/index.txt`; the throwaway `content/_fr/` test artifact was removed afterwards.
* `php -l` and `node --check` pass on the changed files.

## 8.5 Meta translation robustness (added 2026-09-13 after user testing)

**Reported problem:** "quite often when I translate pages with ai, then I get the response 'The AI response for the meta translation was invalid.' and the meta is not translated. It works if I click the update button again."

**Root cause:** the strict meta validation added in 8.1 (previously a silent skip) exposes how often models return the YAML wrapped in markdown code fences or with kept wrapper tags - `Yaml::parse()` rejects those answers. Retrying worked because the model is stochastic.

**Fix in `ControllerApiKixote` (applies to `autotrans` and `autotransUpdate`):**

* New `parseYamlAnswer()` helper: sanitizes the AI answer before parsing - extracts a fenced code block (with or without language tag, also when surrounded by prose) and removes `<article>`/`<translation>` wrapper tag lines. Rejects empty/garbage answers and answers without a `meta:` section (the latter also prevents the AI from accidentally wiping the page meta, since `updateMeta` writes the full document).
* New `getTranslatedMeta()` helper: asks the AI for the meta translation and **retries once automatically** when the first answer does not parse - the manual "click update again" (which re-translates the whole content) is no longer needed. Only after two failed attempts a 400 is returned, with a message clarifying that the content was translated and only the meta failed.
* New `restoreIdentityFields()` helper: after parsing, `pageid`, `translation_for` and `owner` are force-restored from the original meta, so the AI can never change, translate or drop the identity and ownership fields (the prompt already forbids it for pageid/translation_for; now it is also enforced in code).
* The meta system prompt in `autotrans` now also says "Do not wrap the YAML in markdown code fences." (`autotransUpdate` already had the instruction).

**Verification (unit test via reflection, no AI needed):** plain yaml, fenced with/without language tag, prose + fence, and `<article>`/`<translation>`-wrapped answers all parse; empty, prose-only, meta-less and syntactically invalid answers are rejected; `restoreIdentityFields` restores `pageid`/`translation_for`/`owner` while keeping translated values. `php -l` passes.

## 8.6 Missing frontend language links (added 2026-09-14 after user testing)

**Reported problem:** "the german links back to the english version but the english version does not link to the german version (only the first folder)".

**Reproduced against the live install and traced to two root causes:**

1. **Stale extended navigation cache after publishing** (main cause): the publish/unpublish/publishDraft/discard/updateDraft flows in `ControllerApiAuthorArticle` cleared only the draft navigation cache (`clearNavigation([$naviFileName])`, lines 94/201/313/408/514), not the `-extended` cache - unlike create/rename/sort/delete, which clear both. The extended cache stores each page's file path *with extension*; publishing renames `.txt` to `.md`, so the extended cache kept claiming the German pages were drafts. `ControllerWebFrontend::urlIsPublished()` trusted that cached extension, so the English pages did not link to the published German pages. Folder index pages passed because their check reads `index.md` directly from disk - hence "only the first folder" worked. Evidence: `data/navigation/_de/00-loslegen-extended.txt` (mtime 03:07, paths `*.txt`) predates the publishes (03:10:19-29).
2. **Homepages never resolved in `urlIsPublished()`**: `getPageInfoForUrl()` has no home branch (unlike `getItemForUrl()`), so `/` and `/de` always failed the lookup - homepage cross-links and hreflang were missing in both directions.

**Fixes:**

* `ControllerApiAuthorArticle`: all five flows now clear the extended cache too (`clearNavigation([$naviFileName, $naviFileName . '-extended'])`) - consistent with the other content flows; the extension flip `.txt` <-> `.md` is exactly what invalidates the extended cache.
* `ControllerWebFrontend::urlIsPublished()`: now checks the *published state on disk* instead of trusting cached navigation paths (filesystem = source of truth):
  * Homepages are detected directly (`/` and `/lang/`) and checked via `<home>/index.md` on disk - no navigation lookup needed.
  * File pages: the cached extension is ignored; `<path-without-extension>.md` is checked on disk.
  * Folder index pages: unchanged (already a disk check).
  * This also makes the check immune to any already-existing stale extended caches.

**Verification:** re-ran the exact `urlIsPublished` simulation against the live install *with the stale cache still in place*: `/de`, `/`, `/de/loslegen` and all three German subpages now report published in both directions; the untranslated base page (negative control) resolves correctly (no link is generated for it because its index entry is empty). `php -l` passes on both changed files.

Note for the live install: the stale `_de` extended caches are now harmless (the check no longer trusts them) and will be refreshed naturally on the next publish; a one-time "clear navigation" via the Kixote command also cleans them up.