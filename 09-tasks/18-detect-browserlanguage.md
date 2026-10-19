# Detect Browserlanguage

The Cyanine Theme should get an optional function to detect local Browser Language and to redirect users to the correct language version.

## Theme Setting: Detect Browser Language

Add a checkbox to the Typemill theme settings:

- **Label:** `Detect Browser Language`
- **Default:** Disabled
- **Purpose:** Enable or disable automatic language detection and redirection based on the visitor's browser language.

## Behavior

- When enabled, detect the visitor's preferred browser language on their first visit to the website and redirect them to the corresponding language version.
- If the browser language is unsupported, use the language of the base website as the default language.
- Store the detected or manually selected language preference in `localStorage` or a cookie.
- A manually selected language must always take precedence over browser language detection.
- Language switchers must work consistently across all landing pages and subpages and update the stored preference.
- Do not automatically redirect visitors who access language-specific URLs or subpages directly.
- When the setting is disabled, do not perform automatic language detection or redirection. Manual language switching must continue to work.

### Technical Requirements

- Use the existing Typemill theme settings mechanism to register and persist the checkbox value.
- Ensure the detection script runs only when the setting is enabled.
- Prevent redirect loops and avoid overriding explicit language selections.
- Ensure the feature works with cached pages and does not interfere with the existing multilingual routing.

