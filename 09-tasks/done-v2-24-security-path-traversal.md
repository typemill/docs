# Path Traversal in ControllerApiImage::getPagemedia()

**Package:** typemill/typemill
**Tested Versions:** 2.23.1

---

## Affected Files

- `ControllerApiImage.php` — `getPagemedia()` method
- `Storage.php` — `getFolderPath()`, `checkFile()`

---

## Root Cause

`getPagemedia()` passes the user-supplied `path` query parameter directly to `Storage::getFile()` with an empty `$folder` argument. `Storage::getFolderPath()` applies its traversal-prevention regex only when `$folder` is non-empty, so the check is skipped entirely. `Storage::checkFile()` performs only `file_exists()` with no path sanitization.

---

## PoC

```bash
# Read all user credentials/password hashes
curl -u author:password \
  "http://target/api/v1/pagemedia?path=../settings/users"

# Read application config (API keys, secret tokens)
curl -u author:password \
  "http://target/api/v1/pagemedia?path=../settings/settings"
```

---

## Impact

Any Author-level user — the lowest privilege level in Typemill — can read arbitrary `.yaml`, `.txt`, and `.md` files outside the content directory via the `GET /api/v1/pagemedia` endpoint. Confirmed readable targets include `../settings/users` (admin credential hashes) and `../settings/settings` (API keys and secret tokens). Impact is transient read access; no write primitive is demonstrated.

---

## Suggested Remediation

```php
$realpath = realpath($folderpath . $filename);
if (!$realpath || !str_starts_with($realpath, $folderpath)) {
    return false;
}
```

---

## Fix Report

**Status:** Fixed  
**Date:** 2026-06-08  
**File modified:** `system/typemill/Models/Storage.php`

### Changes Made

Added a new `validatePath()` private helper to `Storage.php` that resolves the final file path and enforces directory containment:

- For **existing** files: uses `realpath()` + `str_starts_with()` to verify the resolved path stays within the allowed base directory.
- For **non-existing** files: normalizes the path manually (resolving `..` segments) and performs the same containment check.

Updated the following methods to call `validatePath()` before any filesystem operation:

- `checkFile()` — core fix for the reported `getPagemedia()` exploit
- `getFile()` / `getFileTime()` — read operations
- `writeFile()` / `renameFile()` / `copyFile()` / `deleteFile()` — write/move/copy/delete operations
- `deleteFolder()` / `deleteContentFolder()` / `deleteContentFolderRecursive()` — folder deletion operations

### Verification

- `../settings/users.txt` via `contentFolder` → **BLOCKED** (`Access denied`)
- Deep traversal (`../../../etc/passwd`) → **BLOCKED**
- Normal files inside `content/` → **ALLOWED**

All changes are confined to `Storage.php` (core model) and do not introduce new dependencies or alter controller code.