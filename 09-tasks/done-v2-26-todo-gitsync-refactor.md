# Refactor and Improve the gitsync plugin

Read the concept and implementation from todo-gitsync-plugin.md and apply the following improvements:

1. Change the interface so it looks like in the gitsync_repo_card.html and gitsync_repo_card.png.
2. The configuration fields when edit a repo connection or add a new repo connection should be two fields in one row.
3. Clarify if I can leave the folder path empty if I only want to sync one repo directly in the /content folder
4. If I click on the push or pull, then disable the buttons and show either a spinner or a counter how many files will be synced so admin understands what happens.
5. Can we directly add support for gitlab, too, because it is the most used alternative to github.
6. Is it possible to sync not a whole repository but only certain files or folders?

---

# Review of Refactor Requests

## Status Check

**Design assets found:** `gitsync_repo_card.html` and `gitsync_repo_card.png` exist in `/ai/tasks/`. The design shows a clean, professional card-based layout with:
- Square-corner cards with subtle borders
- Platform icon (GitHub octocat) on the left
- Repo name + branch in header (`typemill/docs (main)`)
- Path mapping info (`github → content/docs`)
- Green "Synced" status badge + last sync metadata (`last pull 7/23/2026, 10:07 PM · 212 files`)
- Teal (`#14b8a6`) Pull and Push buttons on the right
- Small icon-only secondary actions (History, Edit, Delete)
- Expandable sync history panel directly under the card with a data table

---

## Implementation Plan

### 1. Redesign Connection Cards (Visual Overhaul)

**Current state:** Simple bordered box with basic info and action buttons.

**Target:** Match the provided design reference exactly.

**Plan:**
- Connection card uses square corners (`rounded-none` / `border-radius: 0`), matching the design.
- Left side: Platform icon (use other font icons from icomoon, I will add that later, for now add a placeholder icon), repo name, branch in muted color, path mapping line.
- Right side: Pull and Push buttons with **teal background** (`bg-teal-500` / `#14b8a6`), white text, square corners.
- Below path info: Status badge (`Synced` / `Pending` / `Error`) with appropriate color + last sync time and file count.
- Bottom-right of card: Small icon-only buttons for History, Edit, Delete (using placeholder icons, muted color, hover states).
- History panel is **collapsible per card** — clicking the history icon toggles a table directly under that card.
- Use Tailwind classes consistent with Typemill admin. Since Typemill uses `stone` palette, the teal accent (`#14b8a6`) will be added as an inline style or custom class.

**Files to modify:** `js/vue-gitsync.js`

---

### 2. Two-Column Form Layout

**Current state:** Form fields from `gitsync.yaml` render full-width via the standard `component-text` template (`w-full mt-5 mb-5`).

**Plan:**
- In the yaml file you can just add css: lg:w-half to each field definition
- Re-order YAML fields so related fields appear together (e.g. `owner` + `repo`, `branch` + `local_folder`).

**Files to modify:** `gitsync.yaml`

---

### 3. Empty Local Folder (Root Content Sync)

**Current state:** `sanitizeLocalFolder()` rejects empty strings.

**Answer: Yes, with a clear warning.**
- If `local_folder` is empty, files write directly to `/content`.
- **Risk:** Could overwrite the home page `index.md` or collide with existing content.
- **Mitigation:** Allow empty string, show a UI warning: "Syncing to root will place files directly in /content. Existing files may be overwritten."
- Store empty string as `''`. `writeContentFile()` already handles empty `$subFolder` correctly.

**Files to modify:** `gitsync.php` (relax `sanitizeLocalFolder`), `js/vue-gitsync.js` (add warning), `gitsync.yaml` (update description)

---

### 4. Enhanced Loading Feedback with File Counter

**Current state:** Shows a spinner + text like "Pulling from remote...".

**Plan:**
- The backend counts `$filesProcessed` but only returns it at the *end*.
- **Recommended approach (pragmatic):**
  1. Keep the simple request/response model.
  2. Show stage-based status: "Analyzing repository...", "Found N files to sync", "Syncing...".
  3. On completion, show a detailed banner: "Pulled 23 files, deleted 2 files." or "Pushed 8 files to remote."
  4. Include the file count in the response so the UI can display it immediately.

**Files to modify:** `js/vue-gitsync.js`, `gitsync.php` (return detailed counts)

---

### 5. GitLab Support

**Current state:** `GitPlatformClient` abstraction exists with only `GitHubClient`.

**Plan:**
- Create `GitLabClient.php` implementing all 5 abstract methods.
- GitLab REST API v4 endpoints:
  - Auth: `PRIVATE-TOKEN: {token}` header
  - Latest commit: `GET /projects/{id}/repository/commits?ref_name={branch}&per_page=1`
  - Tree: `GET /projects/{id}/repository/tree?recursive=true&per_page=100` (paginated)
  - File: `GET /projects/{id}/repository/files/{path}?ref={branch}` (base64 content)
  - Update: `PUT /projects/{id}/repository/files/{path}` (different payload)
  - Compare: `GET /projects/{id}/repository/compare?from={base}&to={head}&straight=true`
- **Challenge:** GitLab uses numeric project IDs OR URL-encoded `owner/repo` paths.
- **Solution:** Encode `owner/repo` to `urlencode($owner)%2Furlencode($repo)` for the API path.
- Add `gitlab` option to `gitsync.yaml` `platform` select field.

**Files to create:** `GitLabClient.php`
**Files to modify:** `GitPlatformClient.php` (factory), `gitsync.yaml`, `js/vue-gitsync.js`

---

### 6. Selective Sync (Folder/File Filter)

**Current state:** Syncs entire repository.

**Answer: Yes.**

**Plan:**
- Add two optional fields to the connection form:
  - **Source Path** (`repo_path`): Path inside the repo to sync from. Empty = repo root. Example: `docs/`.
  - **File Pattern** (`file_pattern`): Glob pattern. Empty = all `.md`/`.markdown`. Example: `*.md`.
- **Pull logic:**
  - If `repo_path` is set, prepend to API file paths.
  - Strip `repo_path` prefix when computing local path.
- **Push logic:**
  - Only push files that map back into `repo_path`.
  - Prepend `repo_path` to remote path on `updateFile()`.

**Files to modify:** `gitsync.php` (connection CRUD, pull, push, scanLocalMdFiles), `gitsync.yaml` (add fields), `js/vue-gitsync.js` (add fields to formData)

---

## Summary

| # | Task | Complexity | Ready to implement |
|---|------|------------|-------------------|
| 1 | Redesign cards (match design asset) | Medium | ✅ Yes |
| 2 | Two-column form | Low | ✅ Yes |
| 3 | Empty local folder | Low | ✅ Yes |
| 4 | File counter feedback | Medium | ✅ Yes |
| 5 | GitLab support | High | ✅ Yes |
| 6 | Selective sync | Medium | ✅ Yes |

---

(End of file - total 10 lines)