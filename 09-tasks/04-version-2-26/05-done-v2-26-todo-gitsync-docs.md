# GitSync

GitSync connects Typemill to one or more GitHub or GitLab repositories so you can push your content to Git or pull content from Git into Typemill.

## What it does

GitSync keeps your Typemill content and your Git repository in sync:

- **Pull** — download the latest Markdown files from a repository into your Typemill content folder.
- **Push** — upload your Typemill Markdown files to a repository.
- **Status check** — see which files are already in sync, which need a push, which need a pull, and where both sides have changed at the same time.

You can connect several repositories at once. Each repository is mapped to a different folder under `/content`, so you can manage a whole site, a single section, or just one page from Git.

## What you need

- A GitHub or GitLab repository that contains Markdown files.
- A personal access token with write access to the repository contents.
- Typemill 2.25.0 or newer.

## Plugin settings

Open the plugin settings from the system menu.

- **Git Personal Access Token** — the token GitSync uses for all repositories. Store it here once.
  - For GitHub classic tokens, use the `repo` scope.
  - For GitHub fine-grained tokens, grant **Contents** → **Read and write** for each repository.
  - For GitLab, use a token with `api` scope or at least `read_repository` and `write_repository`.
- **API Timeout (seconds)** — how long GitSync waits for the Git platform API before giving up. Increase this only if you sync many large files or have a slow connection.

## Admin dashboard

Go to **System → Gitsync** to open the dashboard.

The dashboard lists every repository connection as a card.

### Repository card

Each card shows:

- **Owner/Repository** — the GitHub or GitLab repository name.
- **Branch** — the branch that is synced.
- **Local folder** — the Typemill folder that receives the files (or `(root)` for `/content`).
- **Source path** — the folder inside the repository that is synced, if you only want a part of the repository.
- **Status badge** — `synced` (green) or `sync now` (red).
- **Last sync** — when the last push or pull happened.

### Actions

- **Pull** — fetch the latest files from the repository and write them to the local folder.
- **Push** — send your local files to the repository.
- **History** — open the history panel for this repository.
- **Edit** — change the connection settings.
- **Delete** — remove the connection. Local files are not deleted.

### Sync status badge

The badge shows the overall state of a repository:

- **Synced** — everything is up to date.
- **Sync now** — at least one file needs a push or pull, or there is a conflict.

Click the badge to open the history panel.

### History panel

The history panel shows two things:

- **Unsynced files** — a table with the current status of each file: `push`, `pull`, or `conflict`.
- **Sync history** — a log of every pull and push, including the time, result, number of files, and message.

## Creating a connection

1. Click **Add repository**.
2. Fill in the fields:

   - **ID** — a unique internal name, for example `docs` or `api`. This cannot be changed later.
   - **Local Folder** — the Typemill subfolder under `/content` where the files are stored. Leave empty to sync directly into `/content`.
   - **File Pattern** — optional glob to filter files, for example `*.md` or `index.md`. Leave empty to sync all Markdown files.
   - **Git Platform** — choose GitHub or GitLab.
   - **API Base URL** — only needed for self-hosted GitLab or GitHub Enterprise. Leave empty for the public platform.
   - **Owner / Organization** — the GitHub/GitLab owner of the repository.
   - **Repository Name** — the repository name.
   - **Branch** — the branch to sync, usually `main`.
   - **Source Path in Repo** — the folder inside the repository that you want to sync. Leave empty for the whole repository.
   - **Auto-sync on frontend** — enable this to let GitSync pull and push this repository automatically while visitors browse the frontend.
   - **Auto-sync interval (minutes)** — minimum time between automatic syncs for this connection (default: 10 minutes).

3. Save the connection.

### Example connections

- Sync a whole repository into `/content/docs`:
  - Local folder: `docs`
  - Owner: `mycompany`
  - Repository: `documentation`
  - Branch: `main`
- Sync only the `docs` folder of a repository into `/content`:
  - Local folder: *(empty)*
  - Owner: `mycompany`
  - Repository: `website`
  - Branch: `main`
  - Source path: `docs`

## Pulling

Pulling downloads the latest files from the repository.

1. Click **Pull** on the repository card.
2. GitSync fetches the latest commit and the changed files.
3. New files are created locally. Modified files overwrite the local version. Removed files are deleted locally.
4. The navigation cache is cleared so the new content appears immediately.

**First pull** — if a connection has never been synced before, GitSync downloads the whole tree of the selected source path.

**Incremental pull** — after the first pull, GitSync only downloads files that changed between the last synced commit and the current commit.

## Pushing

Pushing uploads your local files to the repository.

1. Click **Push** on the repository card.
2. GitSync scans the local folder for Markdown files.
3. Files that changed locally are uploaded to the repository.
4. A new commit is created on the branch.
5. GitSync stores the new commit ID so the status shows synced.

Only files that are tracked or present in the synced folder are pushed. Files that are unchanged since the last sync are skipped.

## Conflicts

GitSync prevents you from accidentally overwriting changes:

- **Pull is blocked** if you have changed a file locally but have not pushed it yet. Push first or resolve the file manually.
- **Push is blocked** if the repository has a newer version of a file that you have not pulled yet. Pull first or resolve the file manually.

A file in a conflict state shows **Resolve conflicts manually before pushing or pulling** under the status badge.

To resolve:

1. Decide which version is correct.
2. Pull the repository version, or edit the local file until it matches the version you want.
3. Push the result.

## Editor meta tab

When you open a page in the visual editor, a **GitSync** tab is added to the meta panel.

The tab shows:

- The repository connection that owns this page.
- The current sync status of that repository.
- A list of unsynced files.
- The sync history for that repository.
- A link to open the GitSync admin dashboard.

If a page is not inside any connected folder, the tab shows that no repository is connected.

## Status states

Each file can be in one of four states:

- **sync** — local and remote are identical.
- **push** — the local file is newer than the tracked remote version.
- **pull** — the remote file is newer than the local version.
- **conflict** — both local and remote have changed since the last sync.

## History

Every pull, push, and error is logged with:

- The time of the action.
- The action type (`pull` or `push`).
- The result (`success`, `error`, or `warning`).
- The number of files affected.
- A short message describing what happened.

The history is stored per repository and kept for the last 20 actions.

## Requirements

- The repository must contain Markdown files (`.md` or `.markdown`).
- The personal access token must have write access to the repository contents.
- The Typemill `/content` folder must be writable so pulled files can be saved.

## Common problems

**Push fails with "Resource not accessible by personal access token"**

The token does not have write access to the repository. Check the token scopes and, for fine-grained GitHub tokens, make sure the repository is selected and **Contents** is set to **Read and write**.

**Pull does nothing**

The repository is already up to date with the last synced commit. Make a change in the repository first, then pull again.

**Status shows "sync now" right after a successful push**

Wait a moment and refresh the status. GitSync now uses the commit ID from the push response directly, so the badge should turn green.

**File appears as pull after I just pushed it**

The Git platform may briefly cache the old commit list. The next status check or refresh will show the correct state.

**The page is not inside any repository**

The local folder of a connection must match the folder that contains the page. For example, a page at `/content/docs/api/page.md` belongs to a connection with local folder `docs` or `docs/api`, not to a connection with local folder `api` or a root connection.

## Best practices

- Use one connection per logical section instead of syncing the whole `/content` folder from a single repository. This makes conflicts and history easier to understand.
- Pull before you start editing, especially if other people or systems also write to the repository.
- Push regularly so your changes are backed up in Git.
- Give each connection a clear ID that matches the folder name, for example `docs` or `api`.
- Use the **Source Path in Repo** if you only want to keep a subfolder of a repository in sync.
