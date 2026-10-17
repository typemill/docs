
## Stored data

Currently you store

```
  file_shas:
      path -> remote blob sha

  file_hashes:
      path -> local content hash
```

These always belong together. Instead you could store

```
files:
    path:
        remote_sha: ...
        local_hash: ...
```

This avoids maintaining two parallel arrays.

Instead of

```
$fileShas[$path]
$fileHashes[$path]
```

you get

```
$files[$path]['remote_sha']
$files[$path]['local_hash']
```

which is easier to maintain and less error-prone.

## Last pull sha 

The only piece that looks unnecessary is having both last_pull_sha and last_push_sha. A single last_sync_sha would express the actual state more clearly.

## Get a repository status overview

In fact, with the metadata you already have, you can compute a very useful status without talking to GitHub except for a single API call.

I would think of each file as having three possible states:

| Local     | Remote    | Status           | Action  |
| --------- | --------- | ---------------- | ------- |
| unchanged | unchanged | ✓ In sync        | nothing |
| changed   | unchanged | ↑ Local changes  | Push    |
| unchanged | changed   | ↓ Remote changes | Pull    |
| changed   | changed   | ⚠ Conflict       | Resolve |

You already have almost everything needed.

Scan all local markdown files and compare

```
sha1(local file)
```

against

```
file_hashes[path]
```

Exactly what your push already does.

Result:

```
localChanged[path] = true
```

No API required.

---

Instead of downloading every file, use GitHub's compare API once.

You already do

```
compareCommits(last_sync_sha, latest_remote_sha)
```

during pull.

The response already contains all changed files.

Build

```
remoteChanged[path] = true
```

No additional requests.

---

Then

```php
foreach ($allFiles as $file) {

    if (!$localChanged && !$remoteChanged)
        status = sync;

    if ($localChanged && !$remoteChanged)
        status = push;

    if (!$localChanged && $remoteChanged)
        status = pull;

    if ($localChanged && $remoteChanged)
        status = conflict;
}
```

That's exactly the information an administrator wants.

Example:

```
✓ index.md

↑ installation.md
  Local changes
  Needs push

↓ quickstart.md
  Updated on GitHub
  Needs pull

⚠ api.md
  Changed locally and remotely
  Conflict
```

---

From those file states you can derive a repository summary:

```
Repository Status

✓ 42 files synchronized

↑ 3 files waiting to push

↓ 5 files waiting to pull

⚠ 1 conflict
```

Then your buttons become obvious:

```
Push (3)

Pull (5)
```

or

```
Resolve Conflict
```

instead of allowing both actions blindly.

---

You can avoid the compare API in one case If

```
last_sync_sha == latest_remote_sha
```

then there are **no remote changes**.

So you don't even need to call `compareCommits()`.

Just compute local changes.

This makes the "status" screen very cheap:

1. `getLatestCommitSha()` (1 request)
2. if identical:

   * scan local hashes
3. if different:

   * call `compareCommits()` once

---

## I'd encapsulate it

I'd create one method like

```php
$status = $this->getRepositoryStatus($connection);
```

returning

```php
[
    'sync' => 42,
    'push' => 3,
    'pull' => 5,
    'conflicts' => 1,
    'files' => [
        [
            'path' => 'docs/index.md',
            'status' => 'sync'
        ],
        [
            'path' => 'docs/install.md',
            'status' => 'push'
        ],
        [
            'path' => 'docs/api.md',
            'status' => 'pull'
        ],
        [
            'path' => 'docs/config.md',
            'status' => 'conflict'
        ]
    ]
];
```

The Vue frontend can then render colored badges, counts, and enable or disable the Push/Pull buttons based on the aggregate state.

I think this fits very naturally into your current design because it reuses the metadata you already maintain rather than introducing any new state. The only structural change I'd still recommend is replacing `last_pull_sha` and `last_push_sha` with a single `last_sync_sha`, since the status calculation only needs to know the last synchronized commit.

---

# Implementation Plan

## Review of the Current State

The plugin is now functional and already covers most of the MVP scope:

- Multi-repository support exists.
- Pull and push are implemented for both GitHub and GitLab.
- Local folder mapping, file patterns, and repo-path filtering exist.
- Navigation cache is cleared after each sync.
- History tracking exists.

However, the metadata structure is still the old one that the review above criticizes. Two concrete changes are needed to realize the improvements in this file:

1. **Combine `file_shas` and `file_hashes` into a single `files` array.**
2. **Replace `last_pull_sha` and `last_push_sha` with a single `last_sync_sha`.**
3. **Add a repository status endpoint that reuses the existing metadata.**

The current data file (`/data/gitsync/gitsync.yaml`) contains only demo data, so a migration is NOT required.

## Implementation Steps

### Phase 1: Metadata Restructuring (Backend)

**1.1. Change the stored schema.**

Current:

```yaml
connections:
    githubdocs:
        ...
        file_shas:
            path.md: abc123
        file_hashes:
            path.md: def456
        last_pull_sha: abc123
        last_push_sha: abc123
```

New:

```yaml
connections:
    githubdocs:
        ...
        last_sync_sha: abc123
        files:
            path.md:
                remote_sha: abc123
                local_hash: def456
```

**1.3. Update all references in `gitsync.php`.**

- `pull()` and `push()` must read/write `connection['files'][$path]['remote_sha']` and `connection['files'][$path]['local_hash']`.
- `last_pull_sha` / `last_push_sha` references become `last_sync_sha`.
- After a successful pull **or** push, store the same `last_sync_sha`.

### Phase 2: Repository Status Endpoint

**2.1. Add a new route.**

```php
[
    'httpMethod'    => 'get',
    'route'         => '/api/v1/gitsync/status/{id}',
    'name'          => 'gitsync.status',
    'class'         => 'Plugins\Gitsync\Gitsync:status',
    'resource'      => 'system',
    'privilege'     => 'read'
]
```

**2.2. Implement `getRepositoryStatus($connection)`.**

The method should return:

```php
[
    'sync'      => 42,
    'push'      => 3,
    'pull'      => 5,
    'conflicts' => 1,
    'files'     => [
        ['path' => 'docs/index.md', 'status' => 'sync'],
        ['path' => 'docs/install.md', 'status' => 'push'],
        ['path' => 'docs/api.md',    'status' => 'pull'],
        ['path' => 'docs/config.md', 'status' => 'conflict']
    ]
]
```

Algorithm:

1. Get `last_sync_sha` from the connection.
2. Scan local synced files (respecting `local_folder`, `repo_path`, and `file_pattern`).
3. For each tracked file, compare `sha1(local_content)` with `connection['files'][$path]['local_hash']`. Build `localChanged[$path] = true`.
4. Fetch the latest remote commit SHA via `getLatestCommitSha()`.
   - If `last_sync_sha === latest_remote_sha`, no remote changes exist.
   - If different, call `compareCommits(last_sync_sha, latest_remote_sha)` once and build `remoteChanged[$path] = true` from the file list.
5. Build status table:

| Local changed | Remote changed | Status   |
|---------------|----------------|----------|
| no            | no             | sync     |
| yes           | no             | push     |
| no            | yes            | pull     |
| yes           | yes            | conflict |

6. Return aggregate counts and the file list.

**2.3. Handle edge cases.**

- First-time sync: if `last_sync_sha` is missing, every local file is either `push` (if new) or `pull` (if tracked remotely). Return a clear `status: 'unknown'` or `first_sync` flag.
- Deleted files: if a tracked file no longer exists locally, treat it as `pull` (will be recreated) unless the remote also removed it.
- New remote files: if not tracked locally, they are `pull`.

### Phase 3: UI Updates (Vue)

**3.1. Fetch status when the Gitsync page loads.**

- Call `GET /api/v1/gitsync/status/{id}` for each connection.
- Cache the result locally, but refresh it after each pull or push.

**3.2. Display status on each connection card.**

- Show a status badge per card:
  - `Synced` (green) when all files are `sync`.
  - `N to push` (blue/amber) when only local changes exist.
  - `N to pull` (blue) when only remote changes exist.
  - `N conflicts` (red) when conflicts exist.
- Disable the **Push** button when there are no local changes or there are conflicts.
- Disable the **Pull** button when there are no remote changes or there are conflicts.
- When conflicts exist, show a prominent `Resolve conflicts` message and explain that manual action is needed.

**3.3. Optional: add a per-connection status detail panel.**

- Expandable list of files with their status (`sync`, `push`, `pull`, `conflict`).
- This helps users understand exactly what will happen before they click a button.

### Phase 4: Testing & Validation

**4.1. Syntax checks.**

- `php -l plugins/gitsync/gitsync.php`
- `node --check plugins/gitsync/js/vue-gitsync.js`

**4.3. Status tests.**

- Edit a local file: status should show `push`.
- Edit a remote file (via GitHub/GitLab UI) and run status: should show `pull`.
- Edit both the same file: should show `conflict`.
- No changes: should show `sync`.

### Phase 5: Cleanup

**5.1. Remove old-key handling.**

- After a reasonable period or after confirming all deployments have migrated, remove the migration helper.

**5.2. Update documentation.**

- Document the new `status` endpoint in the plugin docs if needed.

## Risks & Notes

- **Performance:** The status endpoint scans all local files. For large repositories (hundreds of files) this is still very fast. The remote check is one API call at most.
- **Conflict resolution:** The MVP does not include a merge editor. Conflicts will simply block push/pull until the user manually resolves them outside Typemill.
- **Single SHA vs. dual SHA:** A single `last_sync_sha` is correct as long as every successful pull or push updates it. The push flow already fetches the new remote SHA after committing; this should be written into `last_sync_sha`.

## Files to Modify

- `/plugins/gitsync/gitsync.php` — schema migration, new `status` method, updated pull/push methods.
- `/plugins/gitsync/js/vue-gitsync.js` — status fetching, badge rendering, button state handling.

## Files NOT to Modify

- `/plugins/gitsync/gitsync.yaml` — no form changes needed.
- `/plugins/gitsync/GitHubClient.php` and `/plugins/gitsync/GitLabClient.php` — existing API methods are sufficient.

---

**Decision needed:** I will wait for confirmation before implementing these changes. The metadata migration is a breaking change to the stored YAML format, so it should be reviewed and approved first.

