# Multilanguage Language Page for Cyanine

Yes. I would use a recursive or parent-aware configuration lookup combined with a merge, rather than replacing `settings.themes.cyanine` outright.

Your idea is sound: on every page inside a project, resolve the metadata of that project's homepage and use it to override the relevant Cyanine theme settings. That way, `/it/deeper` inherits the Italian homepage's translations just like `/it`.

There are two separate questions:

1. Can Twig merge the theme settings with the homepage metadata? Yes, depending on the Twig version and available filters.
2. Can Twig find the homepage metadata from any deeper page? Only if the project homepage metadata is accessible through the current template context or you provide a way to look it up. The `project` variable alone doesn't necessarily contain that metadata.

## 1. Merge the settings in Twig

If your Typemill installation has Twig's `merge` filter available, you can do this:

```
{% set themeSettings = settings.themes.cyanine %}
{% set projectSettings = meta.cyhome|default({}) %}

{% set themeSettings = themeSettings|merge(projectSettings) %}
```

The merge order is important: the array passed to `merge` on the right takes precedence when keys collide.

For example, if the theme settings contain:

```
introTitle: Welcome
introMarkdown: Discover my projects.
```

And the Italian homepage metadata contains:

```
introTitle: Benvenuti
```

The merged result contains:

```
introTitle: Benvenuti
introMarkdown: Discover my projects.
```

The Italian title overrides the default, while the default intro text remains.

This is a shallow merge. Since your Cyanine settings are organized into fieldsets, that distinction matters if you later introduce nested configuration structures. Your current fields appear to be largely flat, so a shallow merge may be sufficient.

## 2. Don't overwrite the global settings variable

I would avoid doing this:

```
{% set settings.themes.cyanine = meta.cyhome %}
```

Twig variables are not generally intended to be modified like PHP objects or arrays using arbitrary nested assignments. More importantly, replacing the entire configuration would discard every setting not present in the homepage metadata.

Instead, create a separate effective configuration:

```
{% set cyanine = settings.themes.cyanine|merge(meta.cyhome|default({})) %}
```

Then use `cyanine` in the theme templates:

```
{{ cyanine.introTitle }}
{{ cyanine.footer1 }}
```

Your theme settings remain intact, and the merged configuration is explicit.

## 3. The remaining challenge: finding the project homepage metadata

The example above assumes that `meta.cyhome` is already available in the current page's Twig context. It doesn't solve the lookup for `/it/deeper`.

Your current project detection:

```
{% if '/' ~ project == item.urlRelWoF %}
```

identifies the project homepage when the current page is the project root. But on deeper pages, you need access to the metadata belonging to that root page.

I would handle that lookup in PHP, where Typemill's content tree and page objects can be accessed reliably, and pass the result to Twig as a variable such as `projectHomeMeta`.

Then the Twig code becomes:

```
{% set cyanine = settings.themes.cyanine|merge(projectHomeMeta.cyhome|default({})) %}
```

This keeps the responsibilities clean:

- PHP: resolves the project homepage and retrieves its metadata.
- Twig: merges the metadata with the theme defaults and renders the result.

One detail to verify before implementing this: whether `meta.cyhome` represents a nested metadata object or whether the individual custom fields are exposed directly through `meta`. That determines the exact merge expression.

My preferred approach: resolve the homepage metadata once in PHP for every page belonging to a project, then use the merged `cyanine` variable throughout the theme. This avoids repeating homepage-lookup logic in the footer and every landingpage segment.


## User Suggestions

I have already changed the reference to the theme-data and meta a bit to make it easier and more generic:

{% extends '/layout.twig' %}
{% set theme = settings.themes.cyanine|merge(metatabs.meta.cyhome|default({})) %}
{% set meta = metatabs.meta %}

For the project homepages and their metadata, there is already a feature like this in Typemill:

From the documentation:

> The Twig-function getProjectsHome(settings) returns all meta-data of the startpages from projects. The function is useful if you want to list projects on the homepage or on any other page.

  {% set projectshome = getProjectsHome(settings) %}
 
  <div>
    {% for projectid,projectmeta in projecthome %}
 
      <div class="projectbox">
         <h2>{{projectmeta.meta.title}}</h2>
         <p>{{projectmeta.meta.description}}</p>
         <a href="{{ base_url }}/{{ projectid }}">Read more</a>
      </div>
 
    {% endfor %}
  </div>


So I guess we could do something like this:

  {% set projectshome = getProjectsHome(settings) %}
  {% set cyhome = projectshome[project].metatabs.cyhome %}

---

# Review and Implementation Plan (added by AI, 2026-10-15)

I reviewed the current state of the code. The approach discussed above (merge `settings.themes.cyanine` with the `cyhome` metatab of the current/project homepage, resolve via `getProjectsHome(settings)`) is sound and requires **no PHP/core changes**. Below are my findings, corrections to the snippets above, and a concrete plan.

## Findings

### F1. Bug: wrong merge path in the current `index.twig` (critical)

Current code (uncommitted work in `themes/cyanine/index.twig`):

```
{% set theme = settings.themes.cyanine|merge(metatabs.meta.cyhome|default({})) %}
```

This always merges `{}`. In the stored page yaml, tabs are **top-level siblings**: `meta:`, `cyhome:`, `languageswitch:`, ... (verified against real `content/index.yaml` / `content/_it/index.yaml` files). So the correct path is `metatabs.cyhome`, not `metatabs.meta.cyhome`.

### F2. Correction to the suggested `getProjectsHome` access path

`TwigProjectExtension::getProjectsHome()` returns:

```php
$projects[$id] = ['label' => $label, 'meta' => $meta]; // $meta = parsed content/_<id>/index.yaml
```

So the correct Twig path is `projectshome[project].meta.cyhome` (array key `meta`, then tab key `cyhome`) — not `projectshome[project].metatabs.cyhome`. This matches the doc example (`projectmeta.meta.title`).

### F3. Project homepages never render the landingpage (critical, core behavior)

`ControllerWebFrontend` sets `home = true` only for the URL `/` (line ~92). For a project homepage like `/it`, `Navigation::isHome()` returns true and the homepage item loads, but `home` stays **false**. Consequence: `{% if home and theme.landingpage %}` in `index.twig` never fires on `/it`, `/fr`, ... — the project homepage renders as a normal page (`pages/page.twig`), so the segmented landingpage never appears there regardless of metadata.

Options:
- **(a) Theme-side detection (recommended):** `{% set isHomePage = home or (project and item.urlRelWoF == '/' ~ project) %}` — zero core risk, scoped to cyanine.
- (b) Core change: set `$home = true` for every `isHome($url)`. Semantically cleaner ("project homepages are homepages") but affects **all** themes on all multi-project installations — higher regression risk.

### F4. Empty values would override translations/defaults

The meta API stores whatever the Vue meta form submits (`$metadata[$params['tab']] = $validated['data']`). If an editor fills a cyhome field and later clears it, an **empty string is stored** and a plain `merge` would override the theme default with `''` (e.g. an empty teaser title on `/it`). Fix: filter empty values before merging:

```
{% set theme = settings.themes.cyanine|merge(cyhome|filter((v) => v is not empty)) %}
```

(`filter` with arrow functions requires Twig 3 — we have `twig/twig ^3.11`. Note: Twig's `empty` test treats the string `'0'` as empty; irrelevant for these text fields.)

### F5. Most templates still bypass the merged configuration

Only `index.twig` + `layout.twig` use the merged `theme` variable so far. These still read `settings.themes.cyanine.*` directly and therefore ignore all translations:

- `partials/footer.twig` (footercolumns, footer1-3, copyrightline) — the exact "footer problem" mentioned above
- `partials/breadcrumb.twig` (previous/next labels)
- `partials/navigation.twig` (collapseNav, expand/collapse labels, chapnum)
- `partials/navigationFlat.twig` (chapnum)
- `partials/posts.twig` (blogimage)
- all six `home/*.twig` segment templates (intro/info/teaser/contrast/navi/news)

All of them must switch to `theme.*` for the merge to have any visible effect.

### F6. `404.twig` does not set `theme`

`404.twig` extends `layout.twig` directly, and the 404 pagedata contains no `metatabs`. Once `footer.twig` uses `theme.*`, the footer columns would silently disappear on 404 pages (undefined → `is not empty` fails). Fix: add `{% set theme = settings.themes.cyanine %}` at the top of `404.twig` (no merge needed there; `strict_variables` is off so layout defaults still apply).

### F7. `getProjectsHome` limitations (documented, acceptable)

- Covers only `projectinstances`, **not the base project**. Deeper pages in the base language fall back to theme settings. Acceptable (base language = theme defaults), but should be documented. Optional later improvement: extend `getProjectsHome` to include the base project homepage.
- One uncached yaml read per project instance per page render (`StorageWrapper::getYaml` has no cache). Fine for a handful of languages; revisit only if a site has many projects.

### F8. Slim `cyhome` tab is the right design — with one open question

Restricting `cyhome` to translatable **text** fields (no position numbers, no checkboxes) avoids shallow-merge pitfalls (a stored `false` checkbox or `0` position would otherwise disable/reorder segments per language). Open question: the blog fields (`blog`, `blogfolder`, `blogintro`, `blogimage`) were dropped from `cyhome`, but `blogfolder` is inherently language-specific (`/news` vs `/it/news`). Decide whether to re-add at least `blogfolder` to `cyhome`.

Answer: Yes, re-add blogfolder to cyhome.

### F9. Minor cleanups

- `pages/landingpage.twig` line 47 contains `{{ meta.meta }}` — looks like a debug leftover, should be removed.
- `pages/home.twig` shadows the global `home` flag with a local `{% set home = {...segment positions...} %}`. Harmless (include scope), but renaming it to `segments` would improve clarity. Optional.

## Proposed implementation plan (theme-only, no core changes)

**Step 1 — `themes/cyanine/index.twig`: resolve cyhome correctly and merge**

```
{% extends '/layout.twig' %}

{# effective cyhome config: own homepage meta, or project homepage meta for deeper pages #}
{% set cyhome = metatabs.cyhome|default({}) %}
{% if project %}
    {% set projectshome = getProjectsHome(settings) %}
    {% set cyhome = projectshome[project].meta.cyhome|default(cyhome)|default({}) %}
{% endif %}
{% set theme = settings.themes.cyanine|merge(cyhome|filter((v) => v is not empty)) %}
{% set meta = metatabs.meta %}
{% set isHomePage = home or (project and item.urlRelWoF == '/' ~ project) %}
```

Then use `isHomePage` in the `landingpage`/`blog` branches. Notes:
- The `|default` chains are safe because Twig runs with `strict_variables: false` (verified in `system.php`).
- On the project homepage itself, `metatabs.cyhome` and the `getProjectsHome` lookup return the same data; the lookup also covers deeper pages.
- `{% set %}` in the child runs before `layout.twig` renders, so `theme` is available in the layout and all its includes (established Twig/Typemill pattern).

**Step 2 — switch all remaining templates from `settings.themes.cyanine.*` to `theme.*`**

Files: `partials/footer.twig`, `partials/breadcrumb.twig`, `partials/navigation.twig`, `partials/navigationFlat.twig`, `partials/posts.twig`, `home/landingpageIntro.twig`, `home/landingpageInfo.twig`, `home/landingpageTeaser.twig`, `home/landingpageContrast.twig`, `home/landingpageNavi.twig`, `home/landingpageNews.twig`. Pure search/replace, no logic changes.

**Step 3 — `404.twig`: add theme fallback** (`{% set theme = settings.themes.cyanine %}`).

**Step 4 — cleanup:** remove `{{ meta.meta }}` from `pages/landingpage.twig`.

**Step 5 — test matrix**

| Case | Expected |
|---|---|
| `/` (base homepage) | landingpage with theme settings, optionally overridden by base homepage cyhome |
| `/it` (project homepage) | **landingpage renders** (new), Italian cyhome overrides |
| `/it/deeper` | footer/navigation labels etc. from Italian cyhome |
| page without any cyhome data | unchanged behavior, pure theme settings |
| single-language install (no projects) | unchanged behavior (`project` is null) |
| 404 page | renders with default theme settings incl. footer columns |
| editor: `/it` homepage meta dialog | `cyhome` tab visible (`show: home` + `isHome()` already cover project homepages) |

Clear `cache/twig` if the `twigcache` setting is enabled before testing.

## Open questions (please confirm before implementation)

1. **Project-homepage landingpage rendering (F3):** OK to detect it theme-side via `item.urlRelWoF == '/' ~ project`, or do you prefer the core change so all themes benefit?

Yes, let us try the core change.

2. **`blogfolder` (F8):** re-add blog fields (at least `blogfolder`) to the `cyhome` tab so each language can list its own posts folder?

yes, re-add blogfolder

3. **Empty-string filtering (F4):** OK to drop empty values (so a cleared field falls back to the theme default instead of rendering empty)?

yes, ok.

4. **Scope:** theme-only changes as proposed — no changes to `TwigProjectExtension` or `ControllerWebFrontend` in this task?

yes

I will wait for your confirmation before applying any code changes.
