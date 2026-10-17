# Multilanguage Fixes

There are several errors and improvements for the multilanguage feature: 

1. If I delete any page in the author environment, then the language switch in the frontend disappears. It only appears again if I create (and publish) a new page in a language folder. The same problem happens if I use the Kixote command "clear multilang", it does not recreate the index or at least does not show the language switch in the frontend again. Expected behavior: If a page gets deleted, the frontend switches for all other pages should still be visible and working, so the index should be recreated or updated correctly. Please check related older tasks for implementation details.

2. Regarding the cyanine theme and multilanguage: In the theme settings for the intro segment, add a new checkbox "Show language switch". If activated, then show below the intro button as last element in the intro segment the language switch for each language (e.g. de it fr).

---

# Review & Implementation Plan (added by AI, 2026-10-15)

Status: **PLANNED — waiting for confirmation.**
All findings verified against the current source. Related older task reviewed: `done-v2-27-translation-feature-errors.md` (established the delete-and-lazy-rebuild strategy for the multilang index) and `todo-cyanine-multilanguage-landingpage.md` (established the merged `theme` variable in cyanine).

## 1. Root-cause analysis (verified)

### Problem 1: language switch disappears after page deletion / after "clear multilang"

The invalidation strategy (from the v2-27 translation-fixes task) is: `MultilangExtension` **deletes** `/data/multilang/index.txt` on `onPageCreated`/`onPageDeleted`/`onPageRenamed`/`onPageSorted`, and every consumer is supposed to **lazily rebuild** it when missing.

All API/author consumers do this — `ControllerApiMultilang` (`getMultilangIndex`, `getMultilang`, `createMultilang`, `deleteMultilang`) and `ControllerApiKixote` (`autotrans`, `autotransUpdate`) all contain:

```php
$multilangIndex = $multilang->getMultilangIndex();
if(!$multilangIndex) { $multilangIndex = $multilang->createFreshIndex($this->settings, $urlinfo); }
```

**The frontend controller is the only consumer that does NOT rebuild.** In `ControllerWebFrontend::index()` (`system/typemill/Controllers/ControllerWebFrontend.php:445-503`):

```php
$multilangIndex = $multilang->getMultilangIndex();          // false after deletion
$multilangData  = $multilang->getMultilangData($pageid, $multilangIndex); // null
```

`getMultilangData()` returns `null` for a missing index (`isset(false[$pageid])` → false, no error), so `$multilangRefs` stays empty and `$pagedata['multilang'] = []`. The theme guard `{% if multilang %}` (cyanine `pages/page.twig:156`) is then falsy → **switch and hreflang meta tags silently disappear**. This matches the report exactly:

* After **deleting any page**: `onPageDeleted` deletes the index, the next frontend views find no index → switch gone. It "reappears after creating/publishing a page" only because opening the page in the author area loads the lang tab → `GET /api/v1/multilang` → the API path lazily rebuilds the index. The recreation is accidental, not caused by the page creation itself.
* After **Kixote "clear multilang"**: `ControllerApiGlobals::clearMultilang()` (`ControllerApiGlobals.php:118-129`) only deletes the index; the recreation is lazy and until now only triggered in the author area — never by a frontend pageview.

### Problem 2: cyanine intro language switch (new theme feature)

* Cyanine already renders a language switch from the same `multilang` pagedata in the sidebar (`pages/page.twig:156-162`), so the data source exists and — since the v2-27 homepage fix (8.4) — is also populated on homepages (base and language homes are in the index).
* The intro segment settings live in fieldset `landingpageIntro` in `themes/cyanine/cyanine.yaml` (lines 421-477); the template is `themes/cyanine/home/landingpageIntro.twig`. The intro button block (lines 49-53) is shared by the fullsize and non-fullsize variants, so **one insertion point** (after the button, still inside `.introbg`) covers both.
* The template reads the **merged** `theme` variable (`settings.themes.cyanine` merged with the homepage's `cyhome` meta, `index.twig:12`). Per the design decision in the cyanine-multilanguage-landingpage task, `cyhome` contains translatable **text** fields only — a "show switch" checkbox is a layout decision, not a translation, so the new checkbox lives in the **theme settings only** and is not added to the `cyhome` meta tab.

## 2. Implementation plan

### Fix 1 — lazy rebuild in the frontend controller (`system/typemill/Controllers/ControllerWebFrontend.php`)

In the `MULTILANG` block, replace the direct read with the same lazy-rebuild pattern used by the API controllers, plus a one-time self-heal retry for stale indexes (pageid missing although the index file exists — same "rebuild once, then retry" pattern as `ControllerApiMultilang`/`ControllerApiKixote`):

```php
$multilang      = new Multilang();
$multilangIndex = $multilang->getMultilangIndex();

# index is deleted on every page change (MultilangExtension), so recreate it lazily
if(!$multilangIndex)
{
    $multilangIndex = $multilang->createFreshIndex($this->settings, $urlinfo);
}

$multilangData = $multilangIndex ? $multilang->getMultilangData($pageid, $multilangIndex) : null;

# stale index (existing page missing): rebuild once and retry
if(!$multilangData && $multilangIndex)
{
    $multilangIndex = $multilang->createFreshIndex($this->settings, $urlinfo);
    $multilangData  = $multilangIndex ? $multilang->getMultilangData($pageid, $multilangIndex) : null;
}
```

Then wrap the rest (`unset($multilangData['parent'])`, the project loops, `$pagedata['multilang'] = ...`) in `if($multilangData){ ... }` so a failed rebuild degrades gracefully (no switch, no fatal — same as today when no translations exist).

Notes / tradeoffs:
* This makes the behavior after page deletion, page creation, settings changes **and** "clear multilang" uniform: the index is recreated by the next pageview anywhere (author area **or** frontend). No changes to `MultilangExtension`, the Kixote command, or any API endpoint needed.
* Cost: the first frontend request after an invalidation builds the draft navigation of all languages once and writes the index file; subsequent requests read the file. The retry converges because a fresh index contains every existing page (missing pageids are generated and persisted during the build).
* Known accepted limitation (unchanged, carried over from v2-27): two parallel requests can rebuild simultaneously (last write wins) — acceptable for a derived, deterministic cache.
* Pre-existing quirk noticed but **not** touched (out of scope): slightly broken indentation of the second `foreach($projects ...)` block (lines 480-499); `createFreshIndex` may auto-create a language-home stub on a frontend request — `Navigation::setProject()` already does that on any `/lang/...` URL anyway.

### Fix 2 — cyanine intro language switch (theme-only)

**Step 1 — `themes/cyanine/cyanine.yaml`:** add a checkbox at the end of the `landingpageIntro` fieldset (after `introImage`, lines 474-477), following the existing checkbox style:

```yaml
        introLangswitch:
          type: checkbox
          label: Language Switch
          checkboxlabel: Show language switch below the intro button
          description: Shows links to all published translations of this page. Only visible in multilanguage projects.
```

Not added to the `layouts:` presets (absent = off; presets stay untouched).

**Step 2 — `themes/cyanine/home/landingpageIntro.twig`:** insert after the intro button block (after line 53, before the closing `</div>` of `.introbg`), reusing the data shape and link markup of the existing sidebar switch (`pages/page.twig:156-162`):

```twig
					{% if theme.introLangswitch and multilang %}

						<div class="mt3">
							{% for lang,link in multilang %}
								<a class="link dim ph2 pv1 mh1 ba dib" href="{{ base_url }}{{ link }}">{{ lang }}</a>
							{% endfor %}
						</div>

					{% endif %}
```

Notes:
* `multilang` contains only the **other** languages (the frontend controller skips the active project), consistent with the existing sidebar switch: on the base homepage it renders e.g. `de it fr` — exactly the example in the task. On `/de` it renders `en it fr`.
* The guard `and multilang` keeps the segment clean when the checkbox is on but no published translations exist (or Fix 1's rebuild failed).
* Read via the merged `theme` variable, like all other fields in this template, so the cyhome-merge behavior is unaffected (checkbox is not part of `cyhome`, see analysis above).

### Minor cleanup (optional, recommend same pass)

* Update the Kixote command description in `system/typemill/author/js/vue-kixote.js:82` from "...recreated automatically with the next pageview in the **author area**." to "...recreated automatically with the next pageview." — with Fix 1 this is finally accurate.

## 3. Verification plan (after implementation)

* `php -l` on `ControllerWebFrontend.php`; careful manual syntax check of the twig/yaml edits (no linters in repo); clear `cache/twig` if `twigcache` is enabled before testing.
* Manual QA on a languages-mode installation with 2+ languages and published translations:
  1. Delete any page in the author area → open any other frontend page (base and language) → language switch and hreflang tags still visible and correct; `/data/multilang/index.txt` recreated.
  2. Kixote "clear multilang" → reload a frontend page directly → switch visible again, index file recreated (no author-area visit needed).
  3. Rename/sort a page → frontend switch keeps working on all pages.
  4. Cyanine theme settings → intro segment → enable "Show language switch" → homepage shows the switch as last element below the intro button (both fullsize and non-fullsize intro); disable → gone; non-homepage pages unchanged; pages without translations render nothing extra.
  5. Switch content check: base homepage shows other languages only; a language homepage shows base + other languages, not itself.

## 4. Open questions (please confirm before implementation)

1. **Self-heal retry (Fix 1):** include the "rebuild once + retry" when a pageid is missing from an *existing* (stale) index, or limit this pass to the minimal lazy rebuild when the index file is missing? (Recommend: include — 6 lines, mirrors the API controllers.)
2. **Switch content (Fix 2):** show all languages **except the current one** (consistent with the existing cyanine sidebar switch; recommended), or all languages **including** the current page?
3. Confirm the optional Kixote description cleanup.

I will wait for your confirmation before applying any code changes.

---

# Implementation Report (2026-10-15)

All confirmed items are implemented. 4 files changed, no new files, no new routes/endpoints, no build steps.

## What was changed

### Fix 1 — lazy rebuild + self-heal in the frontend (`system/typemill/Controllers/ControllerWebFrontend.php`, MULTILANG block ~lines 445-531)

* If `Multilang::getMultilangIndex()` returns false (index was deleted by `MultilangExtension` on page create/delete/rename/sort, or via the Kixote "clear multilang" command), the index is now recreated with `Multilang::createFreshIndex($this->settings, $urlinfo)` — the same lazy-rebuild pattern as in `ControllerApiMultilang` and `ControllerApiKixote`. The next pageview anywhere (author area **or** frontend) heals the index.
* Self-heal for stale indexes (confirmed option): if the page's pageid is missing from an *existing* index, the index is rebuilt once and the lookup retried once. A `$indexRebuilt` flag prevents a pointless double rebuild when the first lazy rebuild already ran in the same request (deviation from the API controllers, which may rebuild twice in that edge case).
* Graceful degradation: if the rebuild fails or the page is still not indexed, `unset($multilangData['parent'])` is now guarded by `if($multilangData && is_array($multilangData))`, and the downstream loops simply produce an empty `multilang` array (falsy in Twig) — no switch, no fatal, same as before for missing translations.

### Fix 2 — cyanine intro language switch

* `themes/cyanine/cyanine.yaml`: new checkbox `introLangswitch` at the end of the `landingpageIntro` fieldset (label "Language Switch", checkboxlabel "Show language switch"). Not added to the `layouts:` presets (absent = off) and not part of the `cyhome` meta tab (layout decision, not a translation).
* `themes/cyanine/home/landingpageIntro.twig`: new block after the intro button, as the last element inside `.introbg` — covers both the fullsize and the non-fullsize intro variant. It renders the links from the existing `multilang` pagedata (all languages **except the current one**, confirmed, consistent with the sidebar switch in `pages/page.twig`) and is guarded by `{% if theme.introLangswitch and multilang %}`, so nothing renders when the checkbox is off or no published translations exist.

### Fix 3 — Kixote description cleanup (`system/typemill/author/js/vue-kixote.js:82`)

* "Clear multilang" description now reads "...recreated automatically with the next pageview." (dropped the outdated "in the author area").

## Verification performed

* `php -l` on `ControllerWebFrontend.php`: clean. `node --check` on `vue-kixote.js`: clean.
* `cyanine.yaml` parsed via symfony/yaml: `introLangswitch` present in the `landingpageIntro` fieldset as the last field.
* Twig syntax lint (with stubs for Typemill's custom `markdown` function): `landingpageIntro.twig`, `index.twig`, `pages/page.twig` all pass.
* Runtime smoke test on the live dev install (languages mode: en base + de), which was exactly in the reported bug state (no `/data/multilang/index.txt`):
  1. `getMultilangIndex()` on the missing file → `false` (bug state reproduced — the frontend switch was indeed broken before the fix).
  2. The new lazy-rebuild path built a correct 20-entry index (19 pages + base home `/` linked to German home `/de`) and stored it; serialize roundtrip verified.
  3. `getMultilangData()` returns data for indexed pageids, `null` for unknown ones (the retry-path input).
  4. Content checksums before/after: no churn except `content/_de/index.yaml`, which received its `pageid` + `translation_for` — the documented one-time language-home linking from the v2-27 homepage fix (8.4), required for the homepage switch to work at all.

## Remaining manual QA (needs a running server + browser)

1. Delete any page in the author area → open any other frontend page → language switch and hreflang tags still visible and correct; index file recreated.
2. Kixote "clear multilang" → reload a frontend page directly → switch visible again without visiting the author area.
3. Cyanine theme settings → intro segment → enable "Show language switch" → homepage shows the switch below the intro button (both intro variants); disable → gone; pages without translations render nothing extra.

---

# Follow-up: new bug reported 2026-10-15 (base-level pages cannot be translated)

**Reported:** Base language is de. Create a new page "test" on the base level, open the lang tab, enter a translation url for fr, click create → error "We did not find the base page for this translation." Pages inside folders work; folders on base level also work.

## Root-cause analysis (verified against source + runtime simulation)

The error is thrown in `ControllerApiMultilang::createMultilang()` (`ControllerApiMultilang.php:292-298`) when `getSourceInfo($multilangData[$baselang])` cannot resolve the base page. `getSourceInfo()` (line 667) looks the URL up in `Navigation::getFullExtendedNavigation()` (`Navigation.php:588-624`) — and that function **drops all first-level pages**:

```php
$extended = [];                                  // ← first-level items are never copied into the result
foreach($firstLevelExtended as $key => $item)
{
    if(!$item['path']) continue;
    $extension = pathinfo($item['path'], PATHINFO_EXTENSION);
    if($extension) { continue; }                 // ← "skip files": root-level PAGES are discarded here
    $folderContent = $this->getExtendedNavigation($urlinfo, $langattr, $item['path']);
    $extended      = $extended + $folderContent; // ← only folder subtree caches are merged
}
```

Why folders still work (the asymmetry in the report): each folder-level extended cache (e.g. `data/navigation/02-news-extended.txt`) is generated from a scan that redundantly contains the **first-level items as well** (verified by unserializing the live cache files). So a first-level folder key like `/news` re-enters the result through its own folder cache. A first-level **page** (`/test`, path `/03-test.txt` — has an extension) is skipped, and only re-enters once some folder cache happens to be regenerated after the page was created (stale folder caches don't contain it). Hence: pages on base level always fail right after creation, folders work, and the bug can look intermittent.

Also verified why the page itself is viewable and the lang tab works: the frontend and `getItemForUrl()` resolve URLs via `getPageInfoForUrl()` (`Navigation.php:434-457`), which reads the **first-level** extended navigation directly (no file-skipping) — so this bug is isolated to the multilang create/delete flows (`getSourceInfo`). The same gap exists for pages at the root of a language project (e.g. `/fr/test`), since `getFullExtendedNavigation` uses the same logic for project first-levels.

Consumers of `getFullExtendedNavigation` checked: `ControllerApiMultilang::getSourceInfo()` (real consumer) and `ControllerWebAuthor::showRaw()` (assigns it to an unused variable — dead code, unaffected).

Runtime simulation on this install (in-memory, caches untouched): fresh first-level extended navigation contains `/test`; current logic drops it (`isset($extended['/test']) === false` — bug reproduced); fixed logic keeps it while `/news` + subtree still resolve.

## Proposed fix (one line, in `Navigation::getFullExtendedNavigation`)

Initialize the result with the first-level navigation instead of an empty array:

```php
$extended = $firstLevelExtended;   // was: $extended = [];
```

The rest of the function stays unchanged: files are still skipped for subtree loading (they have none), and `$extended + $folderContent` (union) keeps the **fresh first-level values** for duplicate keys while adding folder subtrees. Effects:

* Root-level pages (base and language projects) resolve in `getSourceInfo()` → create/unlink translation works for them.
* First-level folders now resolve from the fresh root cache instead of a redundant, possibly stale folder-cache copy — strictly better data.
* No new failure modes: worst case (a stale folder cache still listing a deleted first-level page) is the same phantom-entry behavior the current code already exhibits for first-level folders; the multilang flows already handle missing pages gracefully (404 + self-heal, and `copyPage` errors are surfaced).

Side note (optional cleanup, recommend same pass): `content/_de/index.yaml` was auto-linked (`translation_for`) during the earlier index rebuild — that is intended v2-27 behavior, no action needed. Separately, the dead `$extendedNavigation` assignment in `ControllerWebAuthor::showRaw():132` could be removed; not required for this fix.

## Verification plan (after implementation)

* `php -l` on `Navigation.php`.
* Runtime smoke test on this install: recreate a base-level test page, clear the root navi caches, confirm `getSourceInfo('/test')` resolves (via the simulated flow or a real `createMultilang` API call), then remove the test page and clear caches again.
* Manual QA (browser): with base=de and fr instance — create base-level page → lang tab → create fr translation (with and without autotranslate) → succeeds; same for unlink. Regression: translation create/unlink for pages inside folders and for folders still works.

I will wait for your confirmation before applying this fix.

## Implementation Report (2026-10-15)

Applied as proposed — one-line change in `Navigation::getFullExtendedNavigation()` (`system/typemill/Models/Navigation.php:604`):

```php
# start with the first level so that pages on the base level are included (folders are merged with their subtree below)
$extended = $firstLevelExtended;
```

Verification performed:

* `php -l` on `Navigation.php`: clean.
* Regression check with the real function against the live caches: before/after outputs contain the identical 20 keys; the only difference is that first-level folder keys (`/publish-status`, `/news`) now come from the fresh root navigation and appear in natural order instead of via stale folder-cache copies.
* End-to-end test of the exact reported scenario with the real fixed function (run from a throwaway cwd so the live caches stayed untouched: symlinked real `content/`, stale folder caches copied, root caches absent to force fresh regeneration, test page `content/03-test.txt` created): `getFullExtendedNavigation()` now resolves `/test` (path `/03-test.txt`), still resolves `/news` and `/news/fast-website`, 21 keys total (= 20 + `/test`). Test artifacts (`03-test.txt/.yaml`, sandbox) were removed afterwards.
* Note: the multilang index on this install currently contains only the existing content; the fix takes effect for `createMultilang`/`deleteMultilang` immediately (both go through `getSourceInfo` → `getFullExtendedNavigation`).

Remaining manual QA (needs browser): with base=de and fr instance — create a base-level page → lang tab → create fr translation → succeeds; unlink works; regression check for pages inside folders and for folders.