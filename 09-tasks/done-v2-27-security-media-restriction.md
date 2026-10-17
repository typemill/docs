Hi Sebastian,
I'm reporting a security issue privately by email, since the repository
has no security.txt and GitHub private vulnerability reporting is not
enabled. Please do not treat this as public. If you'd rather coordinate
through a private GitHub Security Advisory (I see you already publish
advisories on the repo), open a draft on typemill/typemill and add me as
a collaborator — my GitHub handle is @baeseungwon1010 — and I'm happy to
move the discussion and this PoC there.
This is a follow-up to CVE-2026-71518 (media file-download authorization
bypass, fixed in v2.26.0, commit 8c621063). The fix is effective against
path traversal, but it is incomplete on case-insensitive / Unicode-
normalizing filesystems: the role-restriction ACL and the file that is
actually served are decided by two different notions of "same file", so
a case- or encoding-variant filename bypasses the restriction while the
same protected file is still served — unauthenticated.
Summary
=======
In system/typemill/Controllers/ControllerWebDownload.php (v2.26.0):
- The file to serve is resolved by the filesystem:
    line 45:  $realFilePath = realpath($filepath . $normalizedFilename);
    (followed by the correct prefix-containment check — good against
     traversal.)
- The access-control decision is a raw, case-preserving string-equality
  lookup on the same normalized string:
    line 52:  if($restrictions && isset($restrictions[$filefolder . $normalizedFilename]))
normalizeFilename() rawurldecodes once, rejects NUL and "..", and
collapses slashes — but it does NOT reconcile the string against what
the filesystem considers the same file. On a case-insensitive filesystem
(macOS APFS default, Windows NTFS, or any vfat/exfat/casefolded-ext4
mount used for media/files), realpath() happily resolves "Secret.pdf" to
"secret.pdf", but $restrictions["media/files/Secret.pdf"] is not set, so
the ACL treats it as unrestricted and serves it.
So the restriction key and the served bytes disagree, exactly the class
of "check differs from open" the original CVE was about — just moved from
the path layer to the case/encoding layer.
Affected / threat model (stated honestly)
=========================================
This affects Typemill deployments whose media/files directory lives on a
CASE-INSENSITIVE (or Unicode-normalizing) filesystem:
- macOS (APFS is case-insensitive by default) — common for local/dev and
  some self-hosting;
- Windows (NTFS, case-insensitive by default);
- any Linux host where media/files is a mounted vfat/exfat volume or a
  casefolded ext4 directory.
On a standard case-sensitive Linux ext4 host it does NOT reproduce (the
string used for the ACL and the string the FS resolves are identical).
The attacker is remote and unauthenticated, and needs only to know (or
guess) a restricted media filename and request a case/encoding variant of
it. No credentials, no traversal.
Scope — impact is bounded (I checked for escalation)
====================================================
I audited whether this leads anywhere worse; on the same running
instance it does NOT, and I'm stating that so you can size it accurately:
- Reads stay confined to media/files. The realpath()+containment check
  holds — a symlink planted under media/files pointing at
  settings/settings.yaml resolves outside the base and returns 404; "..".
  is rejected in normalizeFilename() before realpath() is even reached.
- Page access control is NOT affected: it resolves the URL to a single
  cached nav item and reads the restriction from that same item (no
  second independently-keyed lookup), so the case-variant just misses and
  returns 404 — no analogous bypass.
- User password hashes (settings/users/<user>.yaml, bcrypt) sit outside
  media/files and are unreachable from this primitive.
- No unauthenticated write/upload exists to plant a symlink or file
  (upload → login redirect; the file APIs require an authenticated
  author+ role).
So the impact is: unauthenticated disclosure of any *restricted media
file* on a case-insensitive deployment — not a path to credentials, other
protected content, or code execution.
Reproduction (observed on real Typemill v2.26.0)
================================================
Full setup + tests are in the attached poc.sh (benign, localhost only).
Environment: Typemill v2.26.0 (contains the 8c621063 fix), media/files on
a vfat loopback mount (case-insensitive), a real
media/files/filerestrictions.yaml with `media/files/secret.pdf: member`,
and media/files/secret.pdf containing a nonce marker. Unauthenticated
curl:
  A. GET /media/files/secret.pdf   -> 302 Found, Location: /tm/login   (correctly blocked)
  B. GET /media/files/Secret.pdf   -> 200 OK, body: TYPEMILL_SECRET_72e7ee438145807c   (BYPASS)
     GET /media/files/SECRET.PDF   -> 200 OK, same secret bytes
     GET /media/files/%53ecret.pdf -> 200 OK, same secret bytes   (survives normalizeFilename's single rawurldecode)
  C. Same files/config on a case-SENSITIVE ext4 dir:
     GET /media/files/Secret.pdf   -> 404 Not Found   (isolates the divergence to the filesystem)
A is the positive control (the restriction works for the exact key); C is
the filesystem control (no divergence on case-sensitive FS). B is the bug.
Suggested fix
=============
Make the ACL decision use the SAME identity the filesystem uses to open
the file, rather than the raw request string. Options:
- Derive the restriction-map key from the realpath()-resolved path
  (relative to the media base), so the ACL keys on the file actually
  served; or
- Compare case-insensitively / after Unicode normalization when the
  underlying filesystem is case-insensitive; or
- Canonicalize the restriction map and the request through the same
  function that realpath() effectively applies for that filesystem.
Deriving the ACL key from the resolved real path is the most robust — it
closes case, Unicode (NFC/NFD), and any future FS-equivalence variant in
one place.
Verification status
===================
- Observed on real Typemill v2.26.0 with the unmodified vendor controller
  (ControllerWebDownload.php), a real vfat case-insensitive mount, and the
  real filerestrictions.yaml ACL source. Only config/harness was stubbed
  (a dev-server SCRIPT_NAME shim replicating the shipped .htaccess
  front-controller rewrite, and pageaccess:true so the session service
  initializes on the blocked path). The ACL logic under test is vendor
  code.
- Novelty: commit 8c621063 (2026-08-09) is the latest change to
  ControllerWebDownload.php; no later commit addresses case/encoding
  equivalence. Present at HEAD.
AI use disclosure
=================
I used an AI assistant (Claude) for reading the fix diff, standing up the
Typemill instance and the case-insensitive filesystem, and running the
reproduction under my direction. What I did: I identified that the fix
reconciles the path layer but not the case/encoding layer, set the
standard of proof (reproduce on a real case-insensitive filesystem with a
positive control and a case-sensitive control to isolate the cause, using
vendor code unmodified), and validated the result and this report. The
same note is included here for transparency.
Please credit: baeseungwon1010 (GitHub). I have not disclosed this
publicly and will hold public disclosure until a fixed release is
available.
Thanks,
baeseungwon1010

---

# Implementation Plan — Case-/Encoding-Variant Media Download Bypass

## Review Summary

The report is accurate and the vulnerability is real. In
`system/typemill/Controllers/ControllerWebDownload.php` the ACL lookup uses
the URL-normalized filename (`$filefolder . $normalizedFilename`) while
the bytes that are actually served are resolved later via `realpath()`.
On a case-insensitive or Unicode-normalizing filesystem those two strings
can identify the same physical file without being byte-identical, so the
restriction map lookup misses and the protected file is served to
unauthenticated users.

This is a **controller bug** and therefore a permitted change inside
`/system/typemill` per the architecture constraints (bug fixes only).

## Root Cause

Current flow in `ControllerWebDownload::download()`:

1. `normalizeFilename()` decodes the URL and strips traversal sequences.
2. `$restrictions[$filefolder . $normalizedFilename]` is checked for an
   exact string match.
3. `realpath($filepath . $normalizedFilename)` resolves the real on-disk
   path and a prefix check is applied.
4. The file is read and returned.

Step 2 and step 3 use two different identities for the same file. On a
case-insensitive filesystem the ACL key `media/files/Secret.pdf` does not
match the stored restriction `media/files/secret.pdf`, but `realpath()`
resolves to the restricted `secret.pdf` and serves it.

## Proposed Fix

Derive the ACL lookup key from the resolved real path, not from the
request string. This makes the access decision key on the exact file the
filesystem will open, closing case, Unicode-normalization, and any similar
filesystem-equivalence bypasses in one place.

### Concrete Change

In `system/typemill/Controllers/ControllerWebDownload.php`, after the
existing `realpath()` confinement check and before the restriction lookup,
add:

```php
# Build the canonical ACL key from the filesystem-resolved path.
# This ensures the restriction lookup keys on the file actually served,
# not on the request string, which prevents case-/encoding-equivalence
# bypasses on case-insensitive or Unicode-normalizing filesystems.
$canonicalRelativePath = str_replace('\\', '/', substr($realFilePath, strlen($realBasePath) + 1));
$aclKey = $filefolder . $canonicalRelativePath;
```

Then change the restriction lookup from:

```php
if($restrictions && isset($restrictions[$filefolder . $normalizedFilename]))
{
    $allowedrole = $restrictions[$filefolder . $normalizedFilename];
```

To:

```php
if($restrictions && isset($restrictions[$aclKey]))
{
    $allowedrole = $restrictions[$aclKey];
```

Leave all other logic unchanged (prefix containment, MIME detection,
headers, error handling).

### Why This Is the Best Option

- **Same identity for ACL and I/O**: the key is derived from the same
  `realpath()` result that is used to read the file.
- **No filesystem-specific branching**: no need to detect case sensitivity
  or Unicode normalization at runtime.
- **Minimal surface**: one lookup change in a single controller.
- **Backwards compatible on case-sensitive filesystems**: the relative
  path derived from `realpath()` is identical to `$normalizedFilename`, so
  existing restrictions keep working.

## Optional Hardening (Not Required for the Bypass)

1. **Reject directory downloads explicitly**: the existing `validate()`
   method uses `file_exists()`, which is true for directories. Adding an
   `is_file()` check before serving would make the controller more robust
   and avoid accidentally returning a directory with the derived empty
   relative path. This can be bundled with the fix or deferred.
2. **Guard symlink-based parent leakage**: the prefix check is already
   correct; adding a final `!is_link($realFilePath)` check (or resolving
   links before the containment check) would further harden the path. The
   current code is already safe against the reported issue.

## Files to Modify

- `system/typemill/Controllers/ControllerWebDownload.php` (only file
  changed for the fix)

## Verification Steps

1. **Manual reproduction on a case-insensitive filesystem**:
   - Mount `media/files` as vfat/exfat, or test on Windows/macOS with a
     default case-insensitive volume.
   - Create `media/files/secret.pdf` and add
     `media/files/secret.pdf: member` to
     `media/files/filerestrictions.yaml`.
   - Unauthenticated requests:
     - `GET /media/files/secret.pdf` → 302 to login.
     - `GET /media/files/Secret.pdf` → must also 302 to login (was 200).
     - `GET /media/files/SECRET.PDF` → must also 302 to login.
     - `GET /media/files/%53ecret.pdf` → must also 302 to login.

2. **Case-sensitive control**:
   - On a standard ext4 mount, `GET /media/files/Secret.pdf` with no
     `Secret.pdf` file must return 404 (unchanged behavior).

3. **Authenticated happy path**:
   - A user with the allowed role (or a role that inherits it, e.g.
     administrator) must still be able to download the file using the
     exact, mixed-case, or percent-encoded filename.

## Notes / Open Questions Before Implementation

- Does the maintainer want the optional `is_file()` hardening included in
  the same patch, or a strictly minimal one-line lookup change?
- Should a regression test be added? The repository currently has no
  PHPUnit setup, so a reproduction harness (vfat loopback script) can be
  supplied as verification evidence instead.

**Please confirm this plan before I apply the code change.**

---

# Implementation Completed

## Changes Applied

Confirmed by the user. The fix was implemented in
`system/typemill/Controllers/ControllerWebDownload.php`.

Specific changes:

1. **Canonical ACL key derived from `realpath()`**: after the path is
   resolved and confined to `media/files`, the controller now builds the
   restriction-map key from the resolved relative path instead of the raw
   request string. This closes the case-/encoding-equivalence bypass.
2. **Directory downloads explicitly rejected**: added `!is_file($realFilePath)`
   to the existing containment check so directories cannot be served.
3. **Symlink downloads rejected**: added `is_link($requestedPath)` to the
   same check, evaluated against the original requested path (not the
   `realpath()`-resolved target, since `is_link()` on a resolved target
   would miss the link itself).

The `validate()` method was left unchanged; the post-resolution checks
are now authoritative.

## Verification

- `php -l system/typemill/Controllers/ControllerWebDownload.php` → no
  syntax errors.
- A standalone verification script
  (`/tmp/opencode/test_download_security.php`) exercised the canonical-key
  derivation and hardening checks:
  - exact-key restricted file returns 302;
  - case-variant on case-sensitive filesystem returns 404;
  - case-variant on simulated case-insensitive filesystem returns 302
    (bypass closed);
  - percent-encoded case variant returns 302;
  - public unrestricted file returns 200;
  - directory request returns 404;
  - symlink request returns 404;
  - symlink pointing outside the media base returns 404.
- Result: **All checks passed.**

## Remaining Manual Test

Because this environment uses a case-sensitive ext4 filesystem and does not
have privileges to mount a vfat/exfat loopback volume, the actual
case-insensitive filesystem reproduction from the original report should
still be run on a real case-insensitive filesystem (Windows, macOS APFS
default, or a vfat/exfat mount) before release, using the reproduction
steps from the plan above.
