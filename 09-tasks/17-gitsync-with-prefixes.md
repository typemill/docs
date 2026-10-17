# gitsync with prefixes

The GitSync plugin gets out of sync in the following cases:

* If I reorder the files in Typemill and the prefixes (01-) change or a file is moved into a folder, then the files are duplicated when synced with github with the old files keeping the old prefix, so the moved files in Typemill get new files in the repo.
* If I rename files in the repo, then they are also created as new files in Typemill, the old ones are not removed (to be verified).

So the renaming or reordering of files in Typemill and in GitHub results in a duplication, where the old file with the old filename is kept and the renamed or reordered file is created as a new file.

---

## Review & Implementation Plan (added 2026-10-17)

### Root cause analysis (verified in code)

**Push direction (Typemill → repo):** `performPush()` (`plugins/gitsync/gitsync.php:775`) only ever creates/updates remote files.

* `GitPlatformClient` has no `deleteFile()` method — remote deletions never happen at all.
* The tracked-files map (`connections.{id}.files`, keyed by remote path) is used only for hash-based change detection, never to detect local renames/moves/deletions.
* Result: reordering in Typemill changes the `01-` prefixes → files are pushed under their new paths while the old paths stay in the repo forever → duplicates. Same for moving pages into folders and for locally deleted pages.

**Pull direction (repo → Typemill):** `performPull()` (`gitsync.php:408`) mishandles renames.

* GitHub reports renames in the compare API as `status: "renamed"` plus a `previous_filename` field; `GitHubClient::compareCommits()` (`GitHubClient.php:86`) drops `previous_filename`.
* GitLab reports `renamed_file` with `old_path`/`new_path`; `GitLabClient::compareCommits()` (`GitLabClient.php:104`) maps it to status `renamed` but also drops the old path.
* The pull loop treats everything that is not `removed` as "fetch + write", so a remote rename writes the new file locally but never deletes the old local file → duplicates. The "to be verified" case from the task description is therefore **confirmed**.
* Note: renames with <50% similarity (or when the compare API exceeds its file limit) are reported as `removed` + `added`; those already sync correctly on pull.

**Sidecar files:** a Typemill page consists of `.md` + `.yaml` (+ `.txt` for drafts). Core `Content::deletePage()` removes all three, but GitSync's `deleteContentFile()` removes only the `.md`, leaving orphaned `.yaml`/`.txt` files behind when remote deletes/renames are applied locally.

### Implementation plan

**Phase 1 — Client layer: delete support + rename metadata**

1. `GitPlatformClient`: add abstract `deleteFile(string $path, string $message, ?string $sha = null)` returning `['commit_sha' => ...]|false`; extend `makeRequest()` to send a JSON body with `DELETE` and record the HTTP status code (`curl_getinfo(CURLINFO_HTTP_CODE)`) in `$this->lastHttpCode`.
2. `GitHubClient::deleteFile()`: `DELETE /repos/{owner}/{repo}/contents/{path}` with `{message, sha, branch}`. If no sha is passed, fetch it via `getFile()` first; if the file is already gone remotely (404), treat as success.
3. `GitLabClient::deleteFile()`: `DELETE .../repository/files/{encodedPath}` with `{branch, commit_message}`. GitLab answers `204 No Content` with an empty body, so treat an empty response + 2xx status as success.
4. Extend both `compareCommits()` implementations to include `previous_filename` for renames (GitHub: `$file['previous_filename']`; GitLab: `$diff['old_path']` when `renamed_file` is set).

**Phase 2 — Pull: apply remote renames without duplication** (in `performPull()`)

5. Handle `status === 'renamed'` with a `previous_filename`: delete the local file at the old path (plus sidecars, see step 9), unset its entry in the `files` map, then write the new file as today. Evaluate the sync scope (`shouldSyncFile`) per side of the rename: old path in scope + new path out of scope → treat as removal; new path in scope + old out of scope → treat as addition.
6. Conflict pre-scan: for renames, additionally check the *old* local path for unpushed local changes (file exists locally and its hash differs from the stored `local_hash`) → conflict, abort pull. This keeps the existing "never overwrite local work" guarantee.

**Phase 3 — Push: propagate local renames/moves/deletions** (in `performPush()`)

7. After scanning local files, compute **orphans**: tracked files that match the current scope (`repo_path` + `file_pattern`) but no longer exist locally. No explicit rename *pairing* is needed: a reorder/move produces a new local path (pushed by the existing loop) plus an orphaned old path — deleting the orphans is sufficient, and GitHub/GitLab display the resulting delete+create as a rename automatically. This also avoids any mis-pairing risk with duplicate/empty pages.
8. Orphan handling, executed as a **pre-scan before the push loop** to fail fast: for each orphan, `getFile()` the remote path — if 404, just drop it from the map; if the current remote sha differs from the stored `remote_sha`, someone else edited the file → record a conflict and abort with 409 (same semantics as today); otherwise `deleteFile()` and unset from the map. Use the returned `commit_sha` for the `last_sync_sha` bookkeeping (the existing fallback to `getLatestCommitSha()` covers GitLab, which returns no commit sha on delete).
9. Sidecars: extend `deleteContentFile()` to also remove the sibling `.yaml` and `.txt` files (best-effort), mirroring `Content::deletePage()`.
10. Reporting: extend `$detail` with `deleted N` counts (and renamed counts on pull) — the Vue UI already renders `detail`/`message`, so **no JS changes are needed**.

**Phase 4 — Status view (minor, optional)**

11. `getRepositoryStatus()` currently marks a tracked file that is missing locally as "in sync" (`$localChanged[$path] = false`, `gitsync.php:1398`). Show it as `push` (pending deletion) instead so the status view matches the new push behavior.

### Design decisions

* **Scope guard:** remote auto-deletion is limited to files matching the connection's current `repo_path` + `file_pattern`, so out-of-scope repository content is never touched.
* **No rename pairing on push:** delete-orphans + push-new is outcome-equivalent and simpler/safer than hash-based pairing.
* **Non-atomicity:** push already commits file-by-file and can abort mid-way on conflict; the orphan pre-scan keeps the fail-fast behavior for the new delete operations.

### Edge cases considered

* Mass reorder (all prefixes shift at once): unchanged content at new paths is re-pushed (path-keyed hash check can't skip it), old paths are deleted as orphans. Correct, but commit-heavy on first sync after a big reorder — acceptable.
* GitHub compare limits (>300 changed files or <50% rename similarity): arrives as `removed` + `added` → already handled correctly.
* Empty/duplicate pages: no hash pairing → no mis-pairing risk.
* Auto-sync: pull-then-push cycle converges without duplicates once both directions handle renames.

### Test plan (manual, against a test repo)

1. Reorder pages in Typemill → push → repo contains the renamed files, old names gone, no duplicates.
2. Move a page into a folder → push → no duplicates.
3. Delete a page in Typemill → push → removed from repo; if the remote file was edited meanwhile → 409 conflict, nothing deleted.
4. Rename a file in GitHub and in GitLab → pull → old local file incl. sidecars gone, new file present, no duplicates.
5. Remote rename + unpushed local edit of the old file → pull aborts with 409.
6. Enable auto-sync, reorder in Typemill, browse the frontend → converges without duplicates.

### Open points (confirmed 2026-10-17)

* **A. Remote deletion on push:** ✅ confirmed — push deletes orphaned remote files, guarded by a conflict check (remote edited meanwhile → abort with 409) and limited to the connection's `repo_path`/`file_pattern` scope.
* **B. Sidecars:** ✅ confirmed — delete sibling `.yaml`/`.txt` together with the `.md` on pull-deletes/renames (mirrors `Content::deletePage()`).
* **C. Phase 4:** ✅ confirmed — include the small status-view fix.

## Implementation Status (2026-10-17) — DONE

All four phases implemented and verified:

* **Phase 1 (clients):** `GitPlatformClient` gained abstract `deleteFile()`, DELETE-with-body support and HTTP status tracking; `GitHubClient::deleteFile()` (fetches blob sha when needed, treats 404 as success), `GitLabClient::deleteFile()` (handles 204 empty body); both `compareCommits()` implementations now pass `previous_filename` for renames.
* **Phase 2 (pull):** renames delete the old local file (incl. sidecars) and write the new one; conflict pre-scan also guards the old path of a rename.
* **Phase 3 (push):** orphan pre-scan (tracked, in scope, gone locally) with conflict check → remote deletion; fail-fast 409 abort before pushing if an orphan was edited remotely.
* **Phase 4 (status):** tracked files missing locally are now shown as `push` (pending deletion).

**Verification:** `php -l` clean on all 4 changed files; 12 client-layer tests + 26 pull/push logic tests against a stateful mock GitHub/GitLab API server (PHP built-in server, temp scripts in `/tmp/opencode/gitsync-test/`) — all passing, covering: remote rename pull (no duplicates, sidecars removed), rename with local edit → 409, remote delete pull, local reorder push (old path deleted remotely, no duplicates), orphan edited remotely → 409 with nothing pushed, unchanged files skipped.

Remaining: real-world test against an actual GitHub/GitLab repository (test plan above).