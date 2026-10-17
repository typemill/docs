# Authorization Bypass via Path-Normalization Mismatch in Media File Download Route

**Package:** typemill/typemill
**Tested Versions:** 2.25.0 (branch v2.x, commit 8f3901c)

---

## Affected Files

- `system/routes/web.php:67` — `GET /media/files/{params:.*}` route definition; no auth middleware attached
- `ControllerWebDownload::download` — restriction lookup keyed on raw `$params`; file served via `file_get_contents($base.$params)` on the OS-resolved path

---

## Root Cause

`ControllerWebDownload::download` checks `isset($restrictions['media/files/'.$params])` against a restriction map keyed on the raw `$params` string captured from the request path, then serves the file via `file_get_contents($base.$params)` using the OS-resolved path. Path-equivalent variants (`./secret.pdf`, `//secret.pdf`, `%2e/secret.pdf`) are not byte-identical to the restriction key yet resolve to the same file on disk, so the `isset()` check misses while `file_get_contents` succeeds. The `validate()` method blocks only a literal `..` sequence and does not normalize `./`, `//`, or percent-encoded dot segments, leaving all three bypass forms intact.

---

## PoC

1. Run Typemill 2.25.0 on `php:8.3-apache` at `http://localhost:8099`; as admin, restrict `media/files/secret.pdf` to role `editor`.
2. Control (restriction enforced): `GET /media/files/secret.pdf` → `HTTP 302` redirect to login.
3. Bypass: `GET /media/files/./secret.pdf` → `HTTP 200`, file contents returned.
4. Bypass: `GET /media/files//secret.pdf` → `HTTP 200`, file contents returned.
5. Bypass: `GET /media/files/%2e/secret.pdf` → `HTTP 200`, file contents returned.

`poc_typemill_download_bypass.py` (stdlib only) automates the control and three bypass variants. `$_SERVER['REQUEST_URI']` remains raw under Apache, so the bypass is confirmed on both Apache and PHP built-in server deployments.

---

## Impact

An unauthenticated attacker can download any file protected by Typemill's role-based download restriction feature by substituting a path-equivalent URL variant. No credentials, account, or prior access is required; only knowledge of (or a guess at) a restricted file's path. All files an administrator has restricted to a role via the Typemill admin interface are affected. The bypass is repeatable and leaves no distinct log entry separate from a normal request.

---

## Suggested Remediation

1. Canonicalize the requested path before the restriction lookup: collapse `./`, `//`, and percent-decode before keying the `isset()` check, using the same normalized value that is passed to `file_get_contents`.
2. Resolve the final path with `realpath()`, verify it stays within the media root, and apply the role check against the resolved path.
3. Move the role restriction check to route middleware so an unnormalized path cannot reach the controller without passing the gate.
4. Harden `validate()` to reject `./`, `//`, percent-encoded dot segments, and other path-equivalence forms beyond literal `..`.

---

## Implementation Plan Review

### Corrections / Additions to the Task Description

1. **File path correction**: The affected route is defined in `system/typemill/routes/web.php:67`, not `system/routes/web.php`.
2. **Restriction key format**: The controller checks `isset($restrictions['media/files/' . $filename])` against the raw `$filename` string. The YAML file `media/files/filerestrictions.yaml` is stored under the `fileFolder` path (handled by `Storage::getFolderPath('fileFolder')`) and the keys are prefixed with `media/files/`.
3. **No test suite**: The repository does not currently contain PHPUnit or any automated PHP tests. Verification must rely on a manual / scripted HTTP test.

### Proposed Fix

Apply a minimal, defense-in-depth fix inside `system/typemill/Controllers/ControllerWebDownload.php` **only** (this is a security bug fix, not a new feature). Do not modify the route or add middleware; the restriction is per-file and can only be checked after the restriction map is loaded.

#### Step 1: Introduce a path normalization helper

Add a private method (e.g., `normalizeFilename($filename)`) that:

- URL-decodes the path once (`rawurldecode`) to neutralize percent-encoded dot segments (`%2e`, `%2E`).
- Removes a leading slash if present.
- Collapses sequences of two or more slashes (`//`, `///`) to a single slash.
- Removes `./` segments (both as path prefix and in the middle of the path).
- Rejects any remaining `..` segment.
- Returns `false` for any malformed/empty path.

This makes the key used for the restriction lookup identical to the key used for the actual file read.

#### Step 2: Harden `validate()`

Update the private `validate()` method so it validates against the **normalized** filename:

- Ensure the file exists and is readable before returning `true`.
- Keep the existing `..` check but make it segment-based (path component `..`), not just a substring search.
- Reject any path containing `./`, empty segments, leading slashes, or NUL bytes (`\0`).
- Return `false` if the normalized path is `false` or empty.

#### Step 3: Resolve and confine the final path

After normalization and validation, before serving the file:

- Compute `$realFilePath = realpath($filepath . $filename)` and `$realBasePath = realpath($filepath)`.
- If `realpath()` fails or `$realFilePath` does not start with `$realBasePath` followed by a directory separator, abort with a 404 (do not reveal the path).
- Use the real path for `file_get_contents()`, `filesize()`, and `basename()`.

This prevents traversal via symlinks, case/encoding variations, and any other path-equivalence trick that bypasses the string-based restriction key.

#### Step 4: Update restriction lookup

Change the lookup from:

```php
if ($restrictions && isset($restrictions[$filefolder . $filename]))
```

so that `$filename` is the normalized value returned by the helper. The restriction check and the file read must use the same canonical value.

#### Step 5: Update route path note

The route path `system/typemill/routes/web.php` should remain unchanged; the bug is in the controller, not the routing definition.

### Verification Plan

1. Prepare a restricted file:
   - Upload `secret.pdf` to `media/files/`.
   - As admin, restrict `media/files/secret.pdf` to the role `editor`.
2. Run Typemill 2.25.0 on `php:8.3-apache` (or the built-in PHP server) at `http://localhost:8099`.
3. Test the control case (must be blocked):
   - `GET /media/files/secret.pdf` → `302` redirect to login.
4. Test the previously bypassing variants (must now be blocked):
   - `GET /media/files/./secret.pdf` → `404` (or `302` if the path still matches the restriction key; either is acceptable as long as the file is not served).
   - `GET /media/files//secret.pdf` → `404`/`302`.
   - `GET /media/files/%2e/secret.pdf` → `404`/`302`.
   - `GET /media/files/%2e%2e/secret.pdf` → `404`/`302`.
5. Test the legitimate access case:
   - Log in as an `editor` user.
   - `GET /media/files/secret.pdf` → `200` with the PDF content.
6. Test unprotected file access:
   - Upload `public.pdf` and do **not** restrict it.
   - `GET /media/files/public.pdf` → `200` for an unauthenticated user.

### Open Questions / Edge Cases to Confirm

- Does `Storage::getFolderPath('fileFolder')` return an absolute path with a trailing slash? The fix must not break when concatenating `$filepath . $filename`.
- Are there any plugins that rely on the raw `$filename` shape in the download response? The fix only changes internal processing; the URL and the `Content-Disposition` filename remain based on the final real path.
- Should we return `404` instead of `302` for bypass attempts? The current control case returns a `302` redirect when the restriction is hit. The bypass attempts should return **no file content**; returning `404` is safer and leaks less information, but a `302` is also acceptable if the normalized path still matches the restriction key.

### Risks

- **Breaking change**: Any legitimate download URLs that contain `./` or `//` will stop working. This is acceptable; those URLs are unusual and the fix follows standard path canonicalization.
- **Performance**: `realpath()` adds a small filesystem call per request. Negligible for this endpoint.

---

**Status**: Applied on 2026-08-09.

### Investigation Results

- `Storage::getFolderPath('fileFolder')` returns an absolute path with a trailing `DIRECTORY_SEPARATOR` (e.g., `/var/www/html/typemill/media/files/`). The constructor sets `$this->fileFolder = $this->basepath . 'media' . DIRECTORY_SEPARATOR . 'files' . DIRECTORY_SEPARATOR;` and `getFolderPath()` appends another separator after `rtrim()`. Concatenation with the normalized filename is safe.
- Response code for bypass attempts: `404` as requested.

### Applied Changes

File: `system/typemill/Controllers/ControllerWebDownload.php`

- Added `private function normalizeFilename($filename)` to decode percent-encoding, collapse `//`, remove `./`, drop leading slashes, and reject `..` and NUL bytes.
- Updated `download()` to normalize the route parameter before validation, restriction lookup, and file access.
- Added `realpath()` confinement check: the resolved file must be inside the resolved `fileFolder` directory.
- Updated `validate()` to use the normalized filename and the more robust `substr()` endswith check for allowed filetypes.
- Bypass attempts now return `404` with the generic "the requested file does not exist" message.

### Verification Performed

- PHP syntax check passed: `php -l system/typemill/Controllers/ControllerWebDownload.php`.
- Normalization unit tests passed for `./secret.pdf`, `//secret.pdf`, `%2e/secret.pdf`, `../secret.pdf`, `subdir/./secret.pdf`, NUL-byte injection, and valid paths.
- Filesystem confinement tests passed: normalized paths resolve to the expected file within `media/files/`; traversal attempts are rejected.
- Application boot confirmed (controller class loads and route file is unchanged).

### Remaining Manual Verification

The repository has no automated test suite, so a full end-to-end HTTP verification still needs to be performed in a running Typemill instance:

1. Restrict `media/files/secret.pdf` to role `editor`.
2. Unauthenticated `GET /media/files/secret.pdf` → `302`.
3. Unauthenticated `GET /media/files/./secret.pdf`, `/media/files//secret.pdf`, `/media/files/%2e/secret.pdf` → `404`.
4. Logged-in `editor` `GET /media/files/secret.pdf` → `200` with PDF body.
5. Unrestricted `public.pdf` downloadable by unauthenticated users.
