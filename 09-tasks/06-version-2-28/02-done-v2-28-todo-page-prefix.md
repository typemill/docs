# Fix Two Digit Page Prefixes

Typemill currently has a page prefix from 01- to 99- which is limited to 99 pages per folder. If user creates 100 pages, the logic will break.

Evaluate two options:

1. Use a page prefix from 001 to 999 that requires a check and migration for old instances.
2. Keep the current logic with 01 to 99 but add another logic or enhance the logic if user creates page 100 so it still works.

Option 2 would avoid a migration for old instances and there are only a few instances that exceed the 99 pages per folder. Option 2 looks cleaner and more generic. Evaluate both options and create an implementation plan for the prefered solution.

---

## Review & Implementation Plan (2026-10-11)

### Root Cause Analysis

The page prefix is generated in exactly **three places**, all with the same "pad to 2 digits" pattern:

1. `system/typemill/Controllers/ControllerApiAuthorArticle.php:655` (`createArticle()`):
   `$namePath = $index > 9 ? $index . '-' . $slug : '0' . $index . '-' . $slug;`
2. `system/typemill/Models/Storage.php:582` (`moveContentFile()`, used by create/sort/move):
   `$newOrder = ($index < 10) ? '0' . $index : $index;`
3. `system/typemill/Models/Storage.php:1202-1207` (`transformPostsToPages()`): same pattern.

**Important finding: prefix generation and parsing do NOT break at 100 pages.**

- Generation naturally produces `100-slug`, `101-slug`, ... for the 101st+ page (the ternaries only pad small numbers, they don't cap).
- Parsing in `Folder::extractOrderAndSlug()` already accepts any digit count via `/^(\d+)-(.+)$/`.

**The actual breakage is the sort order.** `Folder::scanFolder()` (`system/typemill/Models/Folder.php:17`) relies on `scandir()`, which sorts alphabetically. With a 3-digit prefix, alphabetical order diverges from numeric order. Verified with a PHP simulation:

```
plain sort (scandir):  01-a, 02-b, 09-i, 10-j, 100-x, 101-y, 11-k, 99-z
natural sort:          01-a, 02-b, 09-i, 10-j, 11-k, 99-z, 100-x, 101-y
```

So the 101st page appears between page 10 and 11 in navigation, sitemap and prev/next paging. After any drag&drop sort, `moveContentFile()` renumbers all siblings (`00-`…`100-`…), so the broken mixed-width state persists permanently.

### Option Evaluation

**Option 1 — Migrate everything to `001-`…`999-`:**

- (+) Fixed width, alphabetical `scandir()` keeps working.
- (−) Requires a migration that renames every file/folder of every existing instance (.md/.txt/.yaml triplets + folders). Risky: file permissions, timeouts on shared hosting, interrupted migration leaves a mixed state.
- (−) Only moves the boundary: breaks again at 1000 pages.
- (−) High effort, high risk, touches user content on all instances — while only very few instances are affected.

**Option 2 — Keep `01-`…`99-`, extend gracefully beyond 99:**

- (+) No migration, existing content untouched.
- (+) Code already *mostly* supports it (generation + parsing, see above). Only one real fix needed: numeric-aware sorting in `scanFolder()`.
- (+) Also fixes ordering for legacy mixed-width prefixes (e.g. manually created `1-x` vs `02-y`).
- (+) Minimal change surface (1 required change, 1 optional cleanup).
- (−) None relevant; `strnatcmp` keeps existing order identical for pure 2-digit content.

**Recommendation: Option 2** (matches the leaning in the task description).

### Implementation Plan

#### Step 1 (required): Natural sort in `Folder::scanFolder()`

In `system/typemill/Models/Folder.php`, right after `$folderItems = scandir($folderPath);` (line 17), add:

```php
usort($folderItems, 'strnatcmp');
```

- `strnatcmp` (case-sensitive) stays closest to the current byte-wise `scandir()` order; slugs are lowercased anyway.
- Ordering for all existing ≤99-page instances is unchanged (natural sort == alphabetical sort for equal-width 2-digit prefixes).
- The `.txt`/`.md` pairing logic in the same method is unaffected: files with the same stem remain adjacent under natural sort.
- Posts folders are unaffected: they use fixed-width 12-digit date prefixes, and the descending `rsort($name)` in `getFolderContentDetails()` stays valid.

#### Step 2 (optional, cosmetic): Deduplicate prefix formatting

Add a tiny helper and use it in the 3 generation points listed above to document that prefixes grow beyond 2 digits naturally:

```php
private function formatOrderPrefix($index)
{
    return ($index < 10) ? '0' . $index : (string)$index;
}
```

Behavior identical to today; only removes duplication. (Note: `moveContentFile()` and `transformPostsToPages()` live in `Storage`, `createArticle()` in the controller — the helper would need to exist in both classes or be skipped to keep the change minimal.)

#### Step 3: Verification

1. Create a test content folder with 105 dummy pages (scripted on CLI), e.g. `00-p.md` … `104-p.md`, then scan via the author area: navigation must show them in numeric order.
2. Create the 100th/101st page through the author UI → new pages appear at the end, not between page 10 and 11.
3. Drag&drop-sort items in the 100+ folder → renumbering via `moveContentFile()` keeps correct order.
4. Regression: an existing instance with ≤99 pages per folder shows identical order as before.
5. Check sitemap and prev/next paging on a 100+ folder (both derive from navigation order).
6. Check a posts folder (date prefixes) still sorts descending correctly.

### Edge Cases Considered

- Manually created files via FTP with non-standard prefixes (`1-x`, `0001-y`): natural sort orders them numerically; `extractOrderAndSlug()` parses them. ✔
- Folders (not just files) carry prefixes and go through the same scan. ✔
- `Navigation::renameItem()` preserves the existing prefix string — unaffected. ✔
- `ControllerApiAuthorMeta.php:332` detects posts via `strlen($item->order) == 12` — no collision with 3-digit numeric orders. ✔
- No plugin in this repo calls `scanFolder()`/`moveContentFile()`/`extractOrderAndSlug()` directly. ✔
- No hard-coded `> 99` limits found anywhere in the codebase. ✔

**Status: Implemented (2026-10-11), Option 2 with deduplication helper.**

### Changes Applied

1. `system/typemill/Models/Folder.php` — `scanFolder()`: added `usort($folderItems, 'strnatcmp');` right after `scandir()` so mixed-width numeric prefixes (01- … 99-, 100- …) always sort in numeric order.
2. `system/typemill/Static/Slug.php` — new static helper `Slug::getOrderPrefix($index)`: pads single digits with a leading zero, wider numbers are used as-is.
3. `system/typemill/Controllers/ControllerApiAuthorArticle.php` — `createArticle()`: uses `Slug::getOrderPrefix($index)` instead of the inline ternary.
4. `system/typemill/Models/Storage.php` — `moveContentFile()` and `transformPostsToPages()`: both use `Slug::getOrderPrefix($index)` instead of the inline ternaries (added the `use Typemill\Static\Slug;` import).

### Test Results (CLI, against the real classes via composer autoloader)

- `php -l` on all 4 modified files: no syntax errors. ✔
- Created 105 dummy pages (`00-` … `104-`): raw `scandir()` showed the broken order (`100-` after `10-`), fixed `Folder::scanFolder()` returns all 105 pages in correct numeric order. ✔
- Simulated drag&drop (moved page 104 to the front) on 105 items via `Storage::moveContentFile()`: all siblings renumbered `00-` … `104-`, rescan shows correct new order (`00-page104.md, 01-page0.md, …`). ✔
- Regression — normal folder with ≤12 items incl. subfolders and a `.md`+`.txt` pair: order identical to old behavior, `txtmd` pairing intact. ✔
- Regression — posts folder with 12-digit date prefixes: `rsort()` in `getFolderContentDetails()` still sorts descending correctly. ✔
- No Cypress specs cover page sorting; UI-level check (steps 2/3/5 of the verification plan) still recommended on a dev instance.

