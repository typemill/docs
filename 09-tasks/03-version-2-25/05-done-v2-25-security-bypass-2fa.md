I have identified a vulnerability in Typemill 2.x that allows an attacker to bypass the two-factor authentication (authcode) mechanism entirely. 

I am reaching out privately before any public disclosure to give you the opportunity to patch the issue first. 

Summary: 2FA Authcode Brute-Force Protection Bypass 
Affected version: Typemill 2.x (confirmed on latest commit) 
Severity: Medium 
CWE: CWE-307 (Improper Restriction of Excessive Authentication Attempts) 

The per-user attempt counter (capped at 5) that protects the 6-digit authcode is reset to 0 every time a new authcode is generated — which happens on every fresh login attempt. 

An attacker who knows a valid username and password can therefore cycle login requests indefinitely, resetting the lockout counter each time and gaining unlimited guesses against the 900,000-value keyspace. 

I have confirmed this with a working proof-of-concept on a local Docker instance using the official Typemill Dockerfile. 

I have prepared a full technical write-up including: 

- Root cause analysis with affected file and line references 
- Step-by-step reproduction steps 
- PoC shell script output showing 3 full counter reset cycles 
- Final curl output confirming authenticated session after bypass 
- Suggested fixes 

I am happy to share the full report over email or via a GitHub private security advisory — whichever you prefer.

I plan to follow a standard 90-day disclosure timeline starting from your acknowledgement.

Please let me know how you would like to proceed.


## Fix

Technical assessment

Assuming the description is accurate, the vulnerability sounds plausible.

The attack flow would be:

Attacker knows a valid username and password.
Login triggers generation of a new 2FA code.
Attempt counter starts at 0.
Attacker gets 5 guesses.
Instead of waiting or being locked out, attacker simply performs another login.
New code is generated and counter resets.
Repeat indefinitely.

If that's how Typemill currently behaves, then the rate limiting is effectively tied to the authentication code rather than to the account or login session.

The vulnerability does not bypass 2FA outright—it bypasses the brute-force protection around the second factor. The attacker still has to guess the 6-digit code, but they get unlimited attempts over time.

The simplest fix is to make the failed-attempt counter independent of the authcode.

For example:

Store the failed-attempt counter on the user account (or login session), not on the generated authcode.
Do not reset the counter when issuing a new authcode.
Reset it only after:
successful 2FA,
or after a cooldown period,
or after explicit administrator intervention.

Another option is to refuse issuing a new authcode while the account is locked due to too many failed 2FA attempts.

---

## Implementation Plan

### Affected file
`system/typemill/Controllers/ControllerWebAuth.php`

### Root cause
`storeNewAuthcode()` (line 721) resets the `authcodeattempts` counter to 0 every time a new 6-digit code is generated. A new code is generated on every login attempt that lacks a valid, unexpired device fingerprint. Consequently, an attacker with valid credentials can cycle login requests indefinitely, resetting the brute-force lockout each time and gaining unlimited guesses.

### Strategy
Decouple the per-user failed-attempt counter from the authcode lifecycle. Introduce a lockout timestamp so that 5 failed attempts lock the account for a fixed cooldown period (15 minutes). Do not reset the counter when issuing a new authcode. Clear the lockout only after a successful 2FA validation or after the cooldown expires.

### Step-by-step changes

**1. Remove counter reset on new authcode generation**
- In `storeNewAuthcode()` (currently line 720-721), delete the line:
  `$user->setValue('authcodeattempts', 0);`
- The generated authcode is stored, but the attempt counter stays unchanged.

**2. Add a lockout timestamp field**
- Introduce a new user field `authcodelocked` (Unix timestamp).
- In `loginWithAuthcode()`, when incrementing to the 5th failed attempt (after line 302), set:
  `$user->setValue('authcodelocked', time());`
- Save it together with the incremented attempt counter via the existing `$user->updateUser()`.

**3. Enforce cooldown in `loginWithAuthcode()` before validating the code**
- Immediately after reading `$userdata = $user->getUserData();` (line 256), add an expiry check:
  ```php
  $attempts = $userdata['authcodeattempts'] ?? 0;
  $locked   = $userdata['authcodelocked'] ?? 0;
  if ($attempts >= 5 && $locked && (time() - $locked > 900)) {
      $attempts = 0;
      $user->setValue('authcodeattempts', 0);
      $user->setValue('authcodelocked', 0);
      $user->updateUser();
  }
  ```
- Keep the existing `$attempts >= 5` block that rejects the request when still locked.

**4. Enforce cooldown in `showAuthcodePage()` before generating a new code**
- Before the call to `storeNewAuthcode()` (around line 592), check lockout state:
  ```php
  $attempts = $userdata['authcodeattempts'] ?? 0;
  $locked   = $userdata['authcodelocked'] ?? 0;
  if ($attempts >= 5 && $locked && (time() - $locked <= 900)) {
      return true; // show authcode form but do not generate a new code or send email
  }
  if ($attempts >= 5 && $locked && (time() - $locked > 900)) {
      $user->setValue('authcodeattempts', 0);
      $user->setValue('authcodelocked', 0);
      $user->updateUser();
      // fall through to normal code generation
  }
  ```
- This preserves the existing anti-enumeration behaviour (wrong-password attempts still render the authcode page) while preventing email/code cycling during an active lockout.

**5. Clear lockout on successful 2FA**
- In `loginWithAuthcode()` success path (after line 313), clear the new field together with the attempt counter:
  ```php
  $user->setValue('authcodeattempts', 0);
  $user->setValue('authcodelocked', 0);
  ```

### Verification / testing steps
1. Enable 2FA (`authcode`) for a test user.
2. Submit 5 incorrect authcodes. Expect lockout message on the 5th attempt.
3. Wait < 15 minutes. Perform a fresh username/password login. Expect the authcode page, but no new email. Submit a correct or incorrect code; expect "too many attempts".
4. Wait > 15 minutes. Perform a fresh login. Expect a new authcode email. Submit the correct code. Expect successful login.
5. Submit 5 incorrect authcodes again. Perform a fresh username/password login immediately. Expect the attempt counter NOT to reset (no new email if still locked). Confirm the counter was not reset.
6. Verify that submitting a wrong password still shows the authcode page (no information leak about password correctness).

### Side-effect analysis when 2FA is deactivated
All proposed changes live inside methods that are gated by the authcode feature flag:

- `showAuthcodePage()` exits immediately at line 572 via `isAuthcodeActive()` when `$settings['authcode']` is false or unset. The new lockout logic added after that early-return is never reached, so normal username/password login proceeds unchanged.
- `storeNewAuthcode()` is only called from inside `showAuthcodePage()` after the active check, so removing the attempt-counter reset there has no effect when 2FA is off.
- `loginWithAuthcode()` is the handler for the `/authcode` POST route, which is only registered in `system/typemill/routes/web.php` when `isset($settings['authcode']) && $settings['authcode']` is true. When 2FA is deactivated the route does not exist and this method is never invoked.
- The `authcodeattempts` and `authcodelocked` fields are only read/written inside the authcode-specific methods above. No other controller, model, or middleware references them.

Therefore: **if the 2FA feature is deactivated, the fix is completely inert and cannot interfere with normal login flows.**

### Rollback / compatibility
- Existing users will have `authcodelocked` undefined, which evaluates to `0`. The new logic treats undefined as "not locked", so no data migration is required.
- No new dependencies, routes, or database tables are introduced.
- Changes are confined to a single controller file and follow the existing coding style.

---

**Status:** Implemented in `system/typemill/Controllers/ControllerWebAuth.php`.

**Applied changes:**
1. Removed `authcodeattempts` reset from `storeNewAuthcode()` (line 720-721 deleted).
2. Added `authcodelocked` cooldown expiry check in `loginWithAuthcode()` after reading userdata.
3. Added `authcodelocked` timestamp set on 5th failed attempt in `loginWithAuthcode()`.
4. Cleared `authcodelocked` on successful 2FA in `loginWithAuthcode()`.
5. Added lockout gate in `showAuthcodePage()` before generating a new code: if locked within 15 min, show form but do not send email; if expired, clear lockout and proceed normally.

**Syntax check:** `php -l` passed with no errors.

**When 2FA is deactivated:** all new logic sits behind `isAuthcodeActive()` or the `/authcode` route registration, so it remains completely inert.