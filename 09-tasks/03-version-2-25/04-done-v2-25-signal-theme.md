# Signal Theme for Typemill – Landingpage Refactor Plan

## Task Summary
Refactor the landingpage of the existing `/themes/signal` theme so it matches the HTML prototype in `/themes/signal/typemill-landingpage-signal.html`.

## Constraints & Rules
- All CSS via **Tachyons** first. Custom CSS only if Tachyons cannot achieve it (or is overly complex).
- **All content configurable in theme settings** (`signal.yaml`).
- **No external resources** — all assets (fonts, libraries, images) must be included in the theme.
- **Navigation** must use the current dynamic deep navigation (`navitop.twig`), not hardcoded from the prototype.
- **Footer** must use the current footer (`footer.twig`), do not change its logic, only adjust its style if needed.

---

## Sections to Implement (from prototype)

### 1. Header / Navigation
- Keep existing `layout.twig` header structure.
- Style to match prototype (sticky, border-bottom, logo left, nav center, CTA right).
- Navigation links must come from `navitop.twig` dynamically.
- CTA button in header must be configurable (text + link).

### 2. Hero Section
- Eyebrow text (configurable)
- Headline (configurable, with optional accent line)
- Subtitle (configurable)
- Two CTA buttons (text + link each, configurable)
- Badges/tags below CTAs (configurable list)
- Image card (configurable image upload)

### 3. News Section
- Section eyebrow, title, intro text (configurable)
- List of 3 latest news items (configurable: date + title + link)
- "All news" link (configurable)
- News should auto-pull from a folder (like current `homeNews`)

### 4. Customer Slider / Testimonials
- Section intro (eyebrow, title, text) (configurable)
- Tab buttons (configurable labels)
- Slides (configurable per slide):
  - Group/label (e.g. "Software Manufacturer")
  - Quote text
  - Attribution (name + org)
  - Case study link
  - Image (uploadable)
- 3 Slides maximum.

### 5. Solutions Section
- Section intro (configurable)
- 3 cards (configurable each):
  - Tag/label
  - Title
  - Description
  - Link
  - Featured flag (for first card styling)

### 6. Capability Map
- Section intro (configurable)
- Rows (configurable list of rows):
  - Row title (left side)
  - Tags (right side, configurable list of tags)
- Number of rows should be dynamically
- Tags should support markdown links, so each tag is a link with url and label.

### 7. Ecosystem Section
- Section intro (configurable)
- 3 cards (configurable each):
  - Title
  - Description
  - Link

### 8. Pricing Section
- Section intro (configurable)
- 3 pricing cards (configurable each):
  - Name
  - Cost
  - Tagline
  - Feature list (bullets)
  - CTA button (text + link)
  - Highlight/popular flag

### 10. Getting Started Section
- Section intro (configurable)
- Left: Title, description, two CTAs
- Right: Numbered steps (configurable list)

### 11. Footer
- Keep existing `footer.twig` logic.
- Style to match prototype (border-top, centered, muted text, links).

---

## Files to Create / Modify

### Theme Settings
- `signal.yaml` — Add new fieldsets for all landingpage sections.

### Templates
- `pages/home.twig` — Refactor to include new sections in order.
- `layout.twig` — Adjust header styling and CSS variables if needed.
- `partials/footer.twig` — Style adjustments only.

### New Home Partial Templates
- `home/homeHero.twig`
- `home/homeNews.twig`
- `home/homeSlider.twig` (or `home/homeVoices.twig` renamed)
- `home/homeSolutions.twig`
- `home/homeCapabilities.twig`
- `home/homeCompare.twig`
- `home/homeEcosystem.twig`
- `home/homePricing.twig`
- `home/homeStart.twig`

### Assets
- `css/lume.css` — Add custom CSS where Tachyons is insufficient.
- `js/lume.js` — Add slider/tab JS if needed.
- **Fonts** — Prototype uses `Source Serif 4`, `Inter`, `IBM Plex Mono`. Must include local font files or use system fallbacks.
- **Images** — Any images used in hero, slider, etc. must be stored in theme assets (e.g. `media/` or `images/` within theme).

---

## Open Questions / Ambiguities

1. **Images:** 
   - The image on the stage should be fetched from meta hero image of the homepage
   - The images of the customer stories should be fetched from emta hero image of the linked page.

2. **Fonts:** 
   - Use Tachyons/system font fallbacks (e.g., `georgia` for serif, `system-ui` for sans)
   - Remove the other fonts from prototype

3. **News Section:** 
   - The news list should auto-pull from a folder (like the existing `homeNews` logic)

4. **Section Order:** 
   - The order of sections should be configurable (like current `home` with position numbers) or fixed to match the prototype exactly?

5. **Number of Items:** How many items should be configurable for:
   - News items (add number of items)
   - Slider slides (fixed 3)
   - Capability rows (fixed 6)
   - Pricing cards (fixed 3)
   - Ecosystem cards (fixed 3)

6. **Custom CSS Scope:** 
   - The prototype has extensive custom CSS (grid, hover effects, shadows, transitions). How much of this should be recreated with Tachyons vs. custom CSS? Tachyons does not support CSS Grid, `clamp()`, or many hover transitions out of the box. We will need custom CSS for these.
   - Answer: Use as many tachyons css as possible, if you can solve it without grid, then remove grid. For other options (shadow, transiations) create tachyons like custom css in style.css

7. **Footer Logic:** The current footer supports 1-3 markdown columns + copyright. The prototype shows a simple single-row footer with links and copyright. Should we keep the 3-column footer feature and just style it, or replace it with a simpler single-row footer? The task says "use the current footer, do not change it, only the style." Does this mean we must keep the existing column logic and just style it to look like the prototype (even if the prototype is simpler)?
   - Yes, keep the three column markdown footer.
---

## Proposed Implementation Order

1. Define all theme settings in `signal.yaml` (answer questions above first).
2. Add custom CSS to `lume.css` for grid, hover effects, and prototype-specific styles.
3. Add self-hosted font files if decided.
4. Create each `home/*.twig` partial template.
5. Update `pages/home.twig` to include new partials.
6. Update `layout.twig` header styling and CTA.
7. Style footer to match prototype while keeping existing logic.
8. Test responsiveness and accessibility.

---

## Implementation Report

All tasks have been completed. Below is the summary of what was implemented.

### Theme Settings (`signal.yaml`)
- Added position fields for all new landingpage sections: `heroPosition`, `storiesPosition`, `solutionsPosition`, `capabilitiesPosition`, `comparePosition`, `ecosystemPosition`, `pricingPosition`, `startPosition`.
- Added configurable header CTA: `navCtaText` and `navCtaLink`.
- Added layout width setting: `layoutWidth` with options `landing` (1140px) and `wide` (1280px, original Lume).
- Added new fieldsets for each section with all content fully configurable:
  - `homeHero` — eyebrow, headline, accent, subtitle, 2 CTAs, badges
  - `homeStories` — eyebrow, title, intro, 3 tabbed slides with group, quote, attribution, link, and link label
  - `homeSolutions` — 3 cards with tag, title, text, link, featured flag, and default link label
  - `homeCapabilities` — 6 rows with title and markdown-link tags
  - `homeCompare` — eyebrow, title, intro, markdown table
  - `homeEcosystem` — 3 cards with title, text, link, and link label
  - `homePricing` — 3 cards with name, cost, tagline, features, CTA, highlight flag
  - `homeStart` — eyebrow, title, text, 2 CTAs, numbered steps
- Enhanced `homeNews` with `homeNewsEyebrow` and `homeNewsCount` (configurable number of items).

### Templates Created
| File | Description |
|------|-------------|
| `home/homeHero.twig` | Eyebrow, headline with accent, subtitle, 2 CTAs, badges, hero image from page meta |
| `home/homeStories.twig` | Tabbed customer slider with 3 slides, quotes, attributions, images fetched from linked page meta heroimage |
| `home/homeSolutions.twig` | 3 solution cards with featured styling, tags, hover effects |
| `home/homeCapabilities.twig` | 6 capability rows with markdown link tags |
| `home/homeCompare.twig` | Markdown comparison table |
| `home/homeEcosystem.twig` | 3 ecosystem cards with configurable link labels |
| `home/homePricing.twig` | 3 pricing cards with highlight badge support |
| `home/homeStart.twig` | Getting started band with numbered steps |
| `home/homeNews.twig` | Refactored to list style with dates, hover underline effect, configurable count |

### Templates Modified
| File | Description |
|------|-------------|
| `pages/home.twig` | Added new sections to the sortable array (positions configurable) |
| `layout.twig` | Added landingpage CSS variables, fixed theme paths from `lume` to `signal`, added configurable header CTA, added `--layoutWidth` CSS variable |
| `partials/footer.twig` | Styled to match prototype (border-top, muted text, links) while keeping the 3-column markdown footer logic |
| `signal.yaml` | Added all new landingpage fieldsets, position fields, layout width setting, and `landingpage` readymade. Renamed all `lume` references to `signal`. |

### Assets
| File | Description |
|------|-------------|
| `css/signal.css` | Added extensive landingpage styles: Tachyons-based layout + custom CSS for shadows, hover effects, transitions, responsive grids, and card styling. Updated `.landing-wrap` and `.mw8-5` to use `--layoutWidth` CSS variable. No external resources. |
| `js/signal.js` | Added tabbed slider functionality for customer stories section |

### Key Design Decisions
- **Images:** Hero image from page meta (`meta.heroimage`). Story images fetched from linked page meta heroimage.
- **Fonts:** System/Tachyons fallbacks only (no custom font files). `var(--font-headline)` and `var(--font-body)` are used.
- **News:** Auto-pulls from a configurable folder with configurable item count.
- **Section order:** Configurable via position numbers (same as original theme — set 0 to disable).
- **Custom CSS:** Used Tachyons for layout (flex, grid-like rows). Custom CSS for shadows, transitions, hover effects, and card badges.
- **Footer:** Kept the existing 3-column markdown footer logic (`theme.footercolumns`). Styled to match the prototype's muted look.
- **No external resources:** All CSS and JS are self-contained in the theme.
- **Readymade:** Added a `landingpage` readymade that pre-configures all settings with the original prototype content, text, and section order.

### Responsive Behavior
- All sections use `flex-row-l` / `flex-column` for mobile-first stacking.
- Header CTA hidden on mobile (`display: none` below 768px).
- Comparison table scrolls horizontally on small screens (`overflow-x: auto`).
- Capability rows stack vertically on mobile.
- Customer slider image stacks above text on mobile.

### Accessibility
- Reduced motion support: all transitions disabled when `prefers-reduced-motion` is active.
- Focus-visible outlines added in CSS for links and buttons.
- Semantic HTML (`section`, `header`, `main`, `footer`, `blockquote`).
- Buttons use `aria-label` for burger menu.

