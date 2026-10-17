As an admin, I want to change the design and layout of the authentication page: 

* Change the colors of background and text
* Change the text on the right side
* Optionally add a logo or image on the right side (full markdown content)
* Optionally switch to a stacked layout (top/bottom) instead of the side-by-side layout

The changes should apply to all authentication pages, so authcode.twig, login.twig, recover.twig, recoverconf.twig, reset.twig. Setup.twig is not relevant since this page is not accessible after initial setup.

---

## Implementation Plan

### 1. Goal
Allow admins to customize the visual design and right-side content of all authentication pages (login, authcode, recover, recoverconf, reset) through the existing System Settings form in the admin area.

### 2. Files to Touch

| File | Purpose |
|------|---------|
| `system/typemill/settings/system.yaml` | Reorganize: move redirect fields from `fieldsetaccess` to new `fieldsetauthpage`; add new auth page customization fields |
| `system/typemill/settings/defaults.yaml` | Add default values for the new auth page settings |
| `system/typemill/Controllers/ControllerWebAuth.php` | Pass relevant auth page settings to all auth templates (`login.twig`, `authcode.twig`) |
| `system/typemill/Controllers/ControllerWebRecover.php` | Pass relevant auth page settings to all auth templates (`recover.twig`, `recoverconf.twig`, `reset.twig`) |
| `system/typemill/author/auth/login.twig` | Use settings for colors and right-side content |
| `system/typemill/author/auth/authcode.twig` | Use settings for colors and right-side content |
| `system/typemill/author/auth/recover.twig` | Use settings for colors and right-side content |
| `system/typemill/author/auth/recoverconf.twig` | Use settings for colors and right-side content |
| `system/typemill/author/auth/reset.twig` | Use settings for colors and right-side content |

### 3. New Settings Fields (fieldset `fieldsetauthpage` in `system.yaml`)

Added to the System Settings form under a new tab section "Login". The admin Vue form builder auto-renders these from the YAML definition.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `authpagehint` | `paragraph` | — | Notice explaining what these settings do |
| `authpagebgcolor` | `text` | `#0d9488` | Hex color for the form-side background (left side) |
| `authpagetextcolor` | `text` | `#ffffff` | Hex color for the form-side text and button borders |
| `authpagerightside` | `checkbox` | `true` | Show infos on a right side panel (info / marketing text) |
| `authpagerightbg` | `text` | `#ffffff` | Hex color for the right-side background |
| `authpagerighttextcolor` | `text` | `#000000` | Hex color for the right-side text |
| `authpagerightcontent` | `textarea` | *empty* | Markdown content for the right side of the **login page only**. If empty, the login page shows its default text. If filled, the markdown is rendered on `login.twig` only. Other auth pages always show their original page-specific content. |

> **Sebastian:** `authpagerightcontent` should only apply to the login page. Other auth pages (authcode, recover, recoverconf, reset) have conditional/functional content that must not be overridden.

> **Sebastian:** If authpagerightside is false, use the mobile/stacked layout (content at top, form below).

### 4. Moving Redirect Fields

The three redirect fields (`redirectadminrights`, `redirectcontentrights`, `redirectaccountrights`) are currently under `fieldsetaccess`. They configure where users land **after login**, so they belong in the new "Login" tab. Moving them does not affect stored data because settings are stored as a flat key-value YAML file; the fieldset structure only controls form rendering.

### 5. Passing Settings from Auth Controllers (not Twig Globals)

> **Sebastian:** I do not like this solution and would prefer to add only relevant settings to the twig template in the auth-controller. Any downsides?

**Decision:** Pass only the relevant auth-page settings from the controllers. This keeps the scope minimal and avoids polluting the global Twig context.

**Why this is better:**
- The auth controllers already pass selected settings explicitly (e.g. `recover`, `captcha` in `ControllerWebAuth`). Extending this pattern is consistent.
- No changes to `system.php` are needed.
- Only auth templates receive these variables; no side effects on frontend or admin templates.

**Implementation:**
Add a small private helper method to both `ControllerWebAuth` and `ControllerWebRecover` (or reuse from the base `Controller` if appropriate) that returns the relevant auth-page settings:

```php
protected function getAuthPageSettings(): array
{
    return [
        'authpagebgcolor'        => $this->settings['authpagebgcolor'] ?? '#0d9488',
        'authpagetextcolor'      => $this->settings['authpagetextcolor'] ?? '#ffffff',
        'authpagerightside'      => $this->settings['authpagerightside'] ?? true,
        'authpagerightbg'        => $this->settings['authpagerightbg'] ?? '#ffffff',
        'authpagerighttextcolor' => $this->settings['authpagerighttextcolor'] ?? '#000000',
        'authpagerightcontent'   => $this->settings['authpagerightcontent'] ?? '',
    ];
}
```

Then merge this array into every `$this->c->get('view')->render(...)` call in both controllers that renders an auth template. For example:

```php
return $this->c->get('view')->render($response, 'auth/login.twig', array_merge([
    'recover' => $this->settings['recoverpw'] ?? false,
    'captcha' => $this->settings['authcaptcha'] ?? false,
], $this->getAuthPageSettings()));
```

There are **6 render calls** across the two controllers:
- `ControllerWebAuth`: `login.twig` (1), `authcode.twig` (3)
- `ControllerWebRecover`: `recover.twig` (1), `recoverconf.twig` (1), `reset.twig` (1)

All of them should receive the auth-page settings.

### 6. Template Changes

**Color and layout changes apply to all five auth templates.**

**Common changes in every template:**

1. **Form side (left)** — replace hardcoded `bg-teal-600 text-white` with inline styles driven by `authpagebgcolor` and `authpagetextcolor`. Use the current Tailwind hex values as fallbacks.

2. **Right side colors** — replace hardcoded `lg:bg-white lg:text-black` and `bg-teal-600 text-white` with inline styles driven by `authpagerightbg` and `authpagerighttextcolor`.

3. **`login.twig`**, **`authcode.twig`, `recover.twig`, `recoverconf.twig`, `reset.twig`** — the right side is **never completely hidden** because it contains important dynamic/functional content:
   - If `authpagerightside` is **true**, the right side is rendered as a side panel (`lg:w-1/2`) alongside the form.
   - If `authpagerightside` is **false**, the layout switches to the **stacked/mobile design** on all breakpoints: the right side content is rendered at the top (full width) and the form is below (full width). This is the same behavior the templates already have on mobile (`flex-col-reverse`).

**Different behavior per template:**

4. **login.twig**
   - If `authpagerightcontent` is filled, render it with the `markdown()` Twig function. Otherwise render the original default text ("Welcome back" / homepage link).
   - All other templates always show their original page-specific content (dynamic `authtitle`/`authtext`, recovery instructions, confirmation messages, etc.). `authpagerightcontent` is ignored on these templates.

5. **Button / link color fixes** — remove hardcoded `text-teal-600` on the authcode back-link so it inherits the right-side text color. The submit button in `login.twig`/`recover.twig`/`reset.twig` keeps its `border-stone-50 text-stone-50` classes; if a user chooses a white background they can override the entire right-side content with markdown anyway.


### 7. Default Behavior (Backward Compatibility)

If the admin never touches the new settings, every auth page looks exactly like it does today because the Twig templates use the current hardcoded hex colors as `|default(...)` fallbacks.

### 8. Out of Scope

- `setup.twig` — explicitly excluded per the task description.
- Button hover states — kept as-is; Tailwind's `hover:` classes cannot be overridden dynamically without a custom CSS block, which is beyond the requested background/text color changes.

### 9. Test Steps

1. Open `/tm/system` → the new "Login" fieldset should appear in the settings form. The redirect fields (`redirectadminrights`, `redirectcontentrights`, `redirectaccountrights`) should be visible at the top of it.
2. Change `authpagebgcolor` to `#1a1a1a` and `authpagetextcolor` to `#f0f0f0` → visit `/tm/login` → form side should reflect the new colors.
3. Change `authpagerightbg` to `#f5f5f5` and `authpagerighttextcolor` to `#333333` → right side should reflect the new colors.
4. Check `authpagerightside` off → visit `/tm/login` → the right side panel should move to the top (stacked layout, full width) and the form should be below (full width). The original "Welcome back" text or custom `authpagerightcontent` should appear at the top.
5. Add markdown to `authpagerightcontent` (e.g. an image with `![logo](/media/logo.png)`) → visit `/tm/login` → the login page should display that markdown on the right side. Visit `/tm/recover`, `/tm/reset`, authcode, recoverconf → they should **not** show the custom markdown; they should show their original page-specific content.
6. Clear `authpagerightcontent` → visit `/tm/login` → the login page should show its original default text again.
7. Check `authpagerightside` off → visit `/tm/recover` (or authcode, recoverconf, reset) → the right side should **not** disappear; instead, the right-side content should be at the top (full width) and the form should be below (full width), like the mobile layout.
8. Verify that `setup.twig` is unaffected.
9. Verify that frontend and admin templates do not have the new auth-page variables in their scope (no global pollution).

---

## Decisions & Answers

2. **Should `authpagerightcontent` apply to all auth pages or only login?**  
   Only `login.twig` — Sebastian noted that other auth pages (authcode, recover, recoverconf, reset) contain conditional/functional content that must not be overridden. `authpagerightcontent` is explicitly scoped to the login page only.

3. **Should the `setup.twig` be left completely untouched?**  
   Yes — the task explicitly excludes it.

4. **Should we add translations for the new field labels?**  
   The existing system settings use English labels directly in `system.yaml` and rely on the admin Vue filter `$filters.translate(label)`. If a translation key is missing, the label displays as-is in English. This matches the existing pattern, so no new translation files are needed for the first version.

5. **Should redirect fields move from "Access" to "Login"?**  
   Yes — Sebastian requested this. The redirect fields (`redirectadminrights`, `redirectcontentrights`, `redirectaccountrights`) move to the top of the new `fieldsetauthpage`.

6. **Should `settings` be added to Twig globals or passed from controllers?**  
   Pass from controllers — Sebastian prefers this. A helper method extracts the relevant auth-page settings and merges them into each `render()` call in `ControllerWebAuth` and `ControllerWebRecover`. No changes to `system.php`.

7. **What happens when `authpagerightside` is unchecked?** 
   - The right side is **never hidden** because it contains important dynamic content (verification info, recovery instructions, confirmation messages). Instead, the layout switches to the **stacked/mobile design** (content at top, form below full-width). This ensures no functional content is ever lost.
