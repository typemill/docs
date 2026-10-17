In anticipation of a lot of pages, I've just converted our website to a multi-project one and added a couple of projects.

However, I'm seeing no trace of these projects either on the website itself (I activated the "switch") or in the content management area (no dropdown menu). Perhaps I'm doing something wrong?

    I've checked the "create a multi-project website" box.
    I've checked the "add a project switch to the frontend website" box.
    I've created a label for the "base" website.
    I've added two projects, each with slug (ID) and label.

But nothing has changed on the website...

Any ideas?

Aha! Problem solved! I hadn't actually entered a manual ID for the main/master website – when I entered "base" manually, tada! the relevant dropdown menus appeared. I'm used to Typemill using default values if you don't manually set them, but in this case, it didn't. But it didn't display an error message either!

Just so you know!

---

## Implementation Plan

### Problem Summary
When a user enables multi-project mode (`projects: projects` or `projects: languages`) but leaves the **Base Project ID** or **Base Project Label** fields empty, the system silently fails. `Navigation::checkProjectSettings()` returns `false` because the fields are empty, so no project dropdown appears in the admin area and no project switch appears on the frontend. The user expects default values to be applied automatically, but none are.

### Root Cause
- The form definition in `system/typemill/settings/system.yaml` has `placeholder` values but no `default` values.
- `Validation::recursiveValidation()` does not apply defaults for missing/empty fields.
- `Navigation::checkProjectSettings()` checks `empty($settings['baseprojectid'])` and returns `false` without any feedback to the user.

### Decision: Do NOT auto-fill defaults
After evaluating side effects, we will **not** silently inject `base`/`Base` as defaults.

**Reasons:**
- There is no objectively good universal default. `base` is acceptable for generic multi-project sites, but for `projects: languages` a default of `base` is misleading and confusing.
- Typemill intentionally leaves these fields blank to force an explicit user decision.
- The real bug is the **silent failure**, not the absence of defaults.

### Proposed Fix: Backend validation warning
Add a conditional validation check in the settings save controller so the admin UI surfaces a clear error when multi-project mode is active but base fields are empty.

#### Change in `ControllerApiSystemSettings::updateSettings()` (`/system/typemill/Controllers/ControllerApiSystemSettings.php`)
After `$validatedOutput = $validator->recursiveValidation(...)` succeeds, add:

```php
# Base project fields are mandatory when multi-project is enabled
if (isset($validatedOutput['projects']) && $validatedOutput['projects'] !== 'standard')
{
    if (empty($validatedOutput['baseprojectid']))
    {
        $validator->errors['baseprojectid'] = 'Base Project ID is required for multi-project websites.';
    }
    if (empty($validatedOutput['baseprojectlabel']))
    {
        $validator->errors['baseprojectlabel'] = 'Base Project Label is required for multi-project websites.';
    }
}
```

The existing `if(!empty($validator->errors))` block already returns a 400 with the error messages, and the Vue form builder (`vue-forms.js`) already renders `errors[name]` in red directly under each field. No frontend changes are required.

### Why this approach
- **Zero side effects**: Does not mutate settings, URLs, or filesystem state.
- **Respects user intent**: Keeps the user responsible for defining meaningful IDs/labels.
- **Minimal & safe**: Single 10-line addition in one controller file.
- **Architecture-compliant**: Validation logic stays in the controller right after standard form validation.

### File to modify
1. `system/typemill/Controllers/ControllerApiSystemSettings.php`

### Verification steps
1. In admin system settings, select "Create a multi-project website" (or "multi-language").
2. Leave **Base Project ID** and/or **Base Project Label** empty.
3. Click **Save**.
4. The fields should be highlighted in red with an error message, and `settings.yaml` must not be updated.
5. Fill in both fields and save again — settings should persist normally.

---

**Waiting for confirmation before implementing.**
