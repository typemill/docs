# Improvements of the Soloprint Theme

Theme is located in /themes/soloprint

## Fix the pager left and right in the content area.

Current behavior: The paging icons (pager-arrow) on the left and the right are positioned in the middle of the content area (top:50%), they move down in fixed position if the user scrolls from the middle to the bottom of the page, but they do not move from the middle to the top of the main area. Check the behavior on this page: https://soloprint.typemill.net/getting-started/install-typemill

Expected behavior: The paging icons should be always visible and positioned in the middle of the screen but keep inside the main area. Check the correct and intended behavior in this theme where I copied the elements from: https://books.typemill.net/orgs/intro

## Dark als Standard-Mode

Currently there is a dark mode switch in the theme that always starts with the light mode and can switch to the dark mode with a click.

There should be a new setting (checkbox) with a label like "Default to dark mode". If clicked, the dark mode should be activated for users by default who have not choosen a mode yet.

## Navigation scroll and open

The navigation on the left side is hard to scroll up and down with the mouse, because the scroll balken is not visible and hard to grap with the mouse. It works fine with keys up and down.

If the content structure has folders, there should be an option to open and close the sub-items. Example of how an open/close for navigation items can look like and be implemented can be seen in this theme: https://docs.typemill.net/

---

# Review & Implementation Plan (2026-09-26)

Codebase analysis completed. Root causes identified for all three items. No plan existed before; the plan below is ready for confirmation. **No code has been changed yet.**

## 1. Fix the pager left and right

### Root cause (confirmed)
In `css/style.css` (lines 246–269) the full-height strip uses flexbox centering:

```css
.sidepager { position: absolute; top: 0; bottom: 0; width: 10%; display: flex; align-items: center; ... }
.pager-arrow { position: sticky; top: 50%; ... }
```

`align-items: center` places the sticky arrow's **static position at the vertical middle of the entire page card**. A sticky element with `top: 50%` only starts sticking once its natural position would scroll *above* 50% of the viewport. Therefore on long pages the arrow sits off-screen at content-middle, only follows when you scroll past it, and detaches again at content-middle when scrolling back up — exactly the reported bug.

The reference theme (`publisher` on books.typemill.net) uses identical markup but does **not** vertically center the strip: the arrow's static position is at the **top** of the strip, so sticky pins it to viewport-middle from the first pixel of scrolling, while `top:0; bottom:0` keeps it constrained inside the article.

### Fix (pure CSS, no JS)
- Remove `align-items: center` from `.sidepager` (keep `justify-content: center` for horizontal centering).
- Add `transform: translateY(-50%)` to `.pager-arrow` so the arrow is optically centered on the 50% line.
- Covers both `pages/chapter.twig` and `pages/folder.twig` (shared classes). No Twig changes needed.

## 2. Dark as standard mode

- `soloprint.yaml`: add setting `darkmodedefault: false` and a checkbox field in the existing `elements` fieldset (next to `darkmodebutton`), label "Dark Mode Default", checkboxlabel "Default to dark mode for visitors who have not chosen a mode". Add the key (`false`) to the three readymades for consistency.
- `layout.twig` line 2: render the attribute server-side:
  `data-theme="{{ settings.themes.soloprint.darkmodedefault ? 'dark' : 'light' }}"`
  (Uses `settings.themes.soloprint` directly because `404.twig` extends the layout without setting the `theme` alias.)
- The inline `<head>` script stays as is: a stored `localStorage('soloprint-theme')` choice still wins; visitors without a stored choice get the server-rendered default — exactly the requested semantic.
- `js/script.js` needs no change (toggle reads the current attribute; `aria-pressed` init already derives from it).

Note: enabling the setting flips all *non-choosers* to dark — intended, but worth knowing for existing sites.

## 3. Navigation scroll and open/close

### 3a. Visible, grabbable scrollbar for `.toc-panel`
Add to `style.css`:
- `scrollbar-width: thin` + `scrollbar-color: var(--line) transparent` (Firefox)
- `::-webkit-scrollbar` (~10px), thumb in `var(--line)`, hover in `var(--ink-soft)`, transparent track
- `overscroll-behavior: contain` so page doesn't scroll-chained while navigating the panel

### 3b. Collapsible folders in `partials/toclist.twig`
- Folder items (`elementType == 'folder'` with children actually rendered) get a `<button class="toc-folder-toggle">` with a chevron SVG, `aria-expanded`, and an accessible `aria-label` ("Expand/Collapse *name*"). Button sits in a flex row next to the link (link keeps full navigation behavior, like the reference themes).
- Children `<ul>` gets class `toc-sublist`; collapsed via `.toc-sublist { display: none }` / `.toc-item.open > .toc-sublist { display: block }` (simple, no max-height hacks), chevron rotates via CSS transition.
- Auto-open the path to the current page: add `open` class + `aria-expanded="true"` when `element.active or element.activeParent` (same data cyanine's navigation already uses).
- Guard the toggle with `limit > depth + 1` so the landing page call `toclist.render(navigation, 0, 1, ...)` (no children rendered) stays unchanged.
- Valid HTML: nested `<ul>` stays inside the parent `<li>` (already the case); only `<li>` as direct `<ul>` children.
- `js/script.js`: one delegated click listener on `#toc-panel` toggling `.open` + `aria-expanded` on the closest `.toc-item`. Keyboard support comes free with native `<button>`.

**Decision proposal:** collapse is always on for folders (matches docs.typemill.net), no extra admin checkbox — keeps it simple. 

**Decision** Use the Alternative: gate behind a `collapsiblenav` checkbox if you want the old always-expanded look as an option.

### Drive-by fix (flagged during review)
`partials/toclist.twig` line 14–15 contains malformed HTML: `</span` is missing its closing `>`. Will be fixed while touching the file.

## Files to change
| File | Change |
|---|---|
| `themes/soloprint/css/style.css` | pager fix, scrollbar styling, sublist/toggle styles |
| `themes/soloprint/js/script.js` | folder-toggle delegation |
| `themes/soloprint/partials/toclist.twig` | collapse markup + `</span>` fix |
| `themes/soloprint/soloprint.yaml` | `darkmodedefault` setting, form field, readymades, version bump |
| `themes/soloprint/layout.twig` | server-side `data-theme` |

## Testing / acceptance (manual, no theme test-suite exists)
1. Pager: long chapter page — arrows visible at viewport middle from scroll position 0, stay centered while scrolling, never leave the page card; short page and mobile widths still fine; print view unaffected.
2. Dark default: with cleared localStorage, page loads dark when setting on; toggle still works and persists; with setting off, behavior unchanged.
3. Navigation: panel scrollbar visibly grabbable; folders collapse/expand with mouse and keyboard (Enter/Space, visible focus); folder of the current page auto-opens; landing page contents list unchanged.
4. Version bump to 1.0.1/1.1.0 in `soloprint.yaml` — confirm desired version and whether a new zip in `themes/soloprint/zips/` should be built.

**Awaiting confirmation before applying any code changes.**

---

# Implementation Status (2026-09-26) — DONE

All three items implemented as planned, with the user decision: collapsible navigation is gated behind a new `collapsiblenav` checkbox (default: off = old always-expanded behavior).

## Changes applied

| File | Change |
|---|---|
| `themes/soloprint/css/style.css` | Pager fix: removed `align-items: center` from `.sidepager`, added `transform: translateY(-50%)` to `.pager-arrow` (arrow now sticks to viewport middle from scroll position 0, constrained to the page card). Added visible scrollbar styling for `.toc-panel` (`scrollbar-width/color`, `::-webkit-scrollbar*`, `overscroll-behavior: contain`). Added collapsible-nav styles (`.toc-sublist`, `.toc-folder-toggle` chevron rotation). |
| `themes/soloprint/js/script.js` | Delegated click handler on `#toc-panel` for `.toc-folder-toggle` (toggles `.open` + `aria-expanded`). |
| `themes/soloprint/partials/toclist.twig` | New `collapsible` macro parameter; folder items get a chevron toggle button with `aria-expanded` and escaped `aria-label`; folders containing the active page auto-open via `element.active or element.activeParent`; toggle only rendered when children are actually rendered (`limit > depth + 1`, keeps landing page list unchanged). |
| `themes/soloprint/partials/toc.twig` | Passes `theme.collapsiblenav` to the macro. |
| `themes/soloprint/soloprint.yaml` | New settings `darkmodedefault: false` and `collapsiblenav: false` (+ matching checkbox fields in the "Elements" fieldset, + keys in all three readymades). Version bumped to 1.1.0. |
| `themes/soloprint/layout.twig` | `data-theme` rendered server-side from `settings.themes.soloprint.darkmodedefault`; stored localStorage choice still wins (inline script unchanged). |

## Verification

- Twig syntax lint: all 12 theme templates parse OK.
- 21 functional Twig render assertions passed (collapsible markup, auto-open of active path, limit guard, `data-theme` light/dark, missing-key fallback for existing installs, chapter page integration).
- Live check on the local instance (`/typemill`, soloprint active): with both settings temporarily enabled, `data-theme="dark"` was served and the toc panel rendered 3 folder toggles with the current page's folder auto-open (`aria-expanded="true"`); middle pages render both sidepagers. Settings reverted afterwards — confirmed back to `data-theme="light"` with no toggles (backward compatible for existing installs).
- No zip package was rebuilt; `themes/soloprint/zips/` still contains the 1.0.0 archives.

## How to enable the new features

Admin → Theme settings → Elements: tick **"Default to dark mode"** (`darkmodedefault`) and/or **"Collapsible Navigation"** (`collapsiblenav`). Both are off by default for backward compatibility.