In the ai editor of kixote, long articles are cut at the end, there is probably a max length  for articles in the textarea. please check and increase, so also long texts are fully visible.

---

# Implementation Plan / Review

## Findings

1. **No HTML `maxlength` on the article editor textarea**  
   The Kixote article editor (`system/typemill/author/js/vue-kixote.js` line ~944) uses a plain `<textarea>` bound to `versions[activeversion]` via `v-model`. It has **no `maxlength` / `maxLength` attribute**, so the browser is not artificially cutting text at render time.

2. **No length validation on saving**  
   The draft-update endpoint (`articleUpdate` in `system/typemill/Models/Validation.php`) validates `body` for markdown security but does not enforce a character limit. Long articles can be saved.

3. **AI output token limit is the most likely real cause**  
   - `system/typemill/Controllers/ControllerApiKixote.php` `getOutputBudget()` clamps output tokens to a maximum of **12 000** (default 4000).  
   - `system/typemill/Models/AiAdapter.php` already detects a `finish_reason === 'length'` and reports that the response was cut off.  
   - The system setting `aioutputtoken` (`system/typemill/settings/system.yaml`) currently offers options up to **12000**.
   This means a long AI-generated article can be truncated by the provider before it ever reaches the textarea.

4. **Prompt-body length limit exists, but only for custom prompts**  
   `Validation::kixotePrompt()` limits prompt `content` to 5000 characters. This is unrelated to article text.

5. **CSS/textarea expansion looks correct**  
   The textarea uses `autosize.js` and is inside a scrollable panel (`w-3/4 overflow-auto h-full`). No `max-height` is applied to the editor textarea in `custom.css`.

## Root cause hypothesis

The user-visible symptom (“long articles are cut at the end”) is almost certainly caused by the **AI output token budget**, not by a textarea `maxlength`. The default 4000 tokens (~≈ 3000 words) is quickly exceeded by long articles, and the current cap of 12000 tokens can still be too small for book-length content.

## Proposed changes

1. **Raise the configurable output-token ceiling**  
   - Increase the hard cap in `ControllerApiKixote::getOutputBudget()` from `12000` to a higher safe ceiling (e.g. `64000`).  
   - Add larger option values (e.g. `16000`, `20000`, `32000`, `64000`) to the `aioutputtoken` select in `system/typemill/settings/system.yaml`.  
   - Optionally raise the default from `4000` to a more useful default such as `8000` or `12000`.

2. **Add a hint in the Kixote UI**  
   Add a short note under the article textarea or in the Info tab explaining that very long articles require a higher “Maximum output tokens” setting in System Settings → AI. This helps users self-serve in the future.

3. **(Defensive) Verify textarea display**  
   Confirm that `autosize` is called after `versions` is updated. If any edge-case remains where the textarea does not expand, add a `resizeAiEditor()` call after the AI answer is pushed to `versions` (already present at line ~1431, but worth double-checking).

## Files likely to be modified

- `system/typemill/Controllers/ControllerApiKixote.php` (`getOutputBudget()`)
- `system/typemill/settings/system.yaml` (`aioutputtoken` options)
- `system/typemill/author/js/vue-kixote.js` (UI hint text only)

## Testing plan

1. Open Kixote on a content page with a long article (> 4000 tokens / ~3000 words).
2. Ask Kixote to “write a comprehensive article” or similar.
3. Observe whether the answer is cut off.
4. Increase System Settings → AI → “Maximum output tokens” to a higher value (e.g. 32000).
5. Retry and verify the full text is returned and visible in the textarea.
6. Store the article and confirm it saves completely.

## Clarification from user (2026-09-16)

The user confirmed that the response is complete and the issue is purely **textarea rendering**: the last ~5 lines are visually cut off inside the Kixote article editor. This matches a known autosize + border-box + vertical-padding bug.

## Revised implementation plan

### Root cause

- Tailwind’s preflight sets `box-sizing: border-box` globally.
- The Kixote article textarea (`#kieditor`) uses the `.editor` class which adds vertical padding (`padding: 0.7rem 1.4ch`).
- `autosize.js` can under-calculate the required height when a textarea has vertical padding in border-box mode, causing the last line(s) to be clipped.
- The parent container already provides `p-8` (2rem) padding and the same background color, so the textarea’s own vertical padding is redundant and the source of the miscalculation.

### Proposed changes

1. **Remove vertical padding from `#kieditor`** in `system/typemill/author/css/custom.css`:
   ```css
   #kieditor {
       padding-top: 0;
       padding-bottom: 0;
   }
   ```
   The parent `<div class="... p-8">` keeps the visual spacing (same background color), so layout remains identical while autosize gets an accurate height.

2. **(Defensive) Ensure autosize recalculates reliably** after AI answers are inserted. Verify `resizeAiEditor()` in `system/typemill/author/js/vue-kixote.js` is already called; if the clipped lines persist, wrap the `autosize.update()` call in `requestAnimationFrame` or a tiny `setTimeout` so the browser finishes layout before measuring.

3. **No changes needed to:**
   - AI output token limits
   - `maxlength` attributes (none exist on the article textarea)
   - Article saving/validation (no length limits there)

### Files likely to be modified

- `system/typemill/author/css/custom.css` (primary fix)
- Possibly `system/typemill/author/js/vue-kixote.js` (only if a small timing tweak is needed after testing)

### Testing plan

1. Open Kixote and generate or paste a long article (many paragraphs).
2. Verify the last lines are fully visible without a scrollbar on the textarea itself.
3. Check that the parent panel still scrolls correctly when the whole Kixote panel is taller than the viewport.
4. Confirm the layout and spacing look the same as before (parent `p-8` provides padding).
5. Test in both light and dark mode.

## Status

Confirmed by user.

### Fix 1: textarea rendering (applied)
CSS fix in `system/typemill/author/css/custom.css` removes vertical padding from `#kieditor` so `autosize.js` measures the correct height in Tailwind’s `border-box` layout. The parent container’s `p-8` padding (same background color) preserves the visual spacing.

```css
#kieditor {
  padding-top: 0;
  padding-bottom: 0;
}
```

User confirmed this fix works for a ~15,000 character article.

### Fix 2: AI output-token ceiling and options (applied)
The user then found the AI response itself is truncated by the 12,000 output-token cap.

Changes:
- `system/typemill/Controllers/ControllerApiKixote.php` — raised `getOutputBudget()` ceiling from `12000` to `128000`.
- `system/typemill/settings/system.yaml` — added new `aioutputtoken` options:
  - 16000, 20000, 32000, 64000, 128000 (Max)

### Other reasons checked

- **Provider/model limits:** Native Anthropic Claude models have their own `max_tokens` caps (e.g., 8192 for many models, 128K only with beta headers). If the provider rejects or truncates at its own limit, increasing Typemill’s setting will not help. The user is using an OpenAI-compatible endpoint for Claude Sonnet 5, so the usable limit depends on that proxy.
- **Timeout:** Already configurable up to 600 seconds in System Settings → AI, so it is not a fixed ceiling.
- **Input context:** The 60,000 character limit in `ControllerApiKixote::prompt()` only truncates the optional *example* content, not the main article.
- **Silent truncation UX:** Currently, if a provider returns non-empty content with `finish_reason === 'length'`, Typemill returns the truncated text without warning. This is a separate, potential future improvement (not changed in this task to keep scope minimal).

## Verification

- [x] No `maxlength` or character limit exists on the article textarea.
- [x] Draft saving/validation does not truncate long article bodies.
- [x] CSS textarea clipping fixed.
- [x] AI output-token ceiling raised and more options added.

## Remaining manual testing

1. In System Settings → AI, select a higher token option (e.g., 32000) and save.
2. Open Kixote and generate a long article (~15,000+ characters).
3. Verify the full response is returned and visible in the textarea.
4. If the provider still truncates, try a lower option that matches the provider’s actual limit, or contact the provider about output-token limits.