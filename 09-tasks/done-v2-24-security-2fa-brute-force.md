Dear Typemill Security Team,

My name is Ayan Ambesh. I am an MSc student in Digital Forensics and Information Security at the National Forensic Sciences University (NFSU), Delhi, India. I conduct independent security research on open-source web applications as part of my academic work and responsible disclosure practice.

I am privately reporting a high-severity vulnerability in Typemill CMS v2.23.1 that allows an attacker who knows a user's password to bypass email-based 2FA by brute-forcing the verification code.

I have not published this issue and will follow a 90-day coordinated disclosure timeline.

Summary
- Severity: High
- CVSS 3.1: 7.4 (AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N)
- CWE-338: Use of Cryptographically Weak Pseudo-Random Number Generator
- CWE-307: Improper Restriction of Excessive Authentication Attempts
- Affected version: Typemill v2.23.1
- Main affected file: system/typemill/Controllers/ControllerWebAuth.php

Technical details
1) Weak code generation (CWE-338)
In ControllerWebAuth.php, auth codes are generated using PHP rand() with a 5-digit range:

private function generateAuthcodeValue()
{
    return rand(10000, 99999);
}

This provides only 90,000 possible values and uses a non-cryptographic PRNG.

2) Missing rate limiting (CWE-307)
The POST /tm/authcode route in system/typemill/routes/web.php accepts unlimited attempts per session with no lockout or cooldown.

Attack scenario
- Attacker logs in with valid username/password (e.g., stolen credentials).
- Session enters 2FA-pending state.
- Attacker brute-forces all 90,000 codes against POST /tm/authcode.
- Correct code is accepted; attacker is redirected into the admin panel.

Proof of concept result
- Environment: local Docker Typemill v2.23.1
- Tool: async Python PoC (aiohttp)
- Concurrency: 100 workers
- Throughput: about 200 requests/second
- Success: valid code found in 184.9 seconds after 36,978 attempts
- Verified outcome: admin panel access confirmed (HTTP 200)

MailHog verification
The code discovered by brute force matched the 2FA email code captured in MailHog during the same session.

Security impact
This vulnerability allows complete bypass of 2FA once the password is known, which defeats 2FA as a protection layer and enables unauthorised admin access.

Recommended remediation
1) Replace rand() with random_int() and increase to 6 digits:

private function generateAuthcodeValue()
{
    return random_int(100000, 999999);
}

2) Add strong server-side protections on POST /tm/authcode:
- Limit attempts per session/account/IP
- Add cooldown after failures
- Invalidate code after N failed attempts
- Force full re-authentication after lockout

3) Long-term: consider moving to TOTP (RFC 6238) instead of email OTP.

Disclosure timeline
- 2026-06-07: Vulnerability identified
- 2026-06-08: PoC verified on local Typemill v2.23.1
- 2026-06-08: Private disclosure sent
- 2026-09-06: 90-day public disclosure deadline

I am happy to provide additional technical detail, test a fix, and help validate the patch before release.

Thank you for your time and for maintaining Typemill.

Best regards,
Ayan Ambesh
MSc Digital Forensics and Information Security
National Forensic Sciences University (NFSU), Delhi, India

Attachments
1. 01_vulnerable_code.png
2. 03_poc_running.png
3. 04_2fa_email.png
4. 05_mailhog_inbox.png
5. 06_login_page.png
6. poc_typemill.py

---

## Fix Report (2026-06-08)

Applied a minimal patch to 3 files to mitigate the reported brute-force vulnerability.

### Changes

**1. Stronger, longer codes (`system/typemill/Controllers/ControllerWebAuth.php`)**
- Replaced `rand(10000, 99999)` with `random_int(100000, 999999)`.
- Fixes CWE-338 (weak PRNG) and expands the search space from 90,000 to 1,000,000 combinations.

**2. Per-user attempt counter in the user profile**
- Added `authcodeattempts` field to the user YAML.
- Counter is reset to `0` every time a new code is generated.
- After **5 wrong attempts** the endpoint hard-rejects further tries for that code (attacker must re-authenticate with password to get a new code).
- On a wrong code the counter is incremented and the user file is saved immediately.

**3. Delays to slow brute force**
- `usleep(300000)` (0.3 s) on every `/tm/authcode` submission before validation.
- `usleep(200000)` (0.2 s) after generating a new code to slow rapid code cycling.

**4. 6th digit field**
- `system/typemill/author/auth/authcode.twig`: added `code-6` input (existing JS auto-focus already handles it).
- `system/typemill/Models/Validation.php`: requires and validates `code-6` with the same 1-digit regex.

### Impact on the reported attack
- The PoC achieved ~200 req/s. The 0.3 s sleep alone drops that to ~3 req/s.
- Combined with the 5-attempt limit, a single code can only be hit 5 times before lockout.
- 6 digits mean 1,000,000 combinations instead of 90,000.
- In practice the attack is no longer feasible in the 5-minute code window.

### Files modified
- `system/typemill/Controllers/ControllerWebAuth.php`
- `system/typemill/author/auth/authcode.twig`
- `system/typemill/Models/Validation.php`

No new middleware, no database, no external storage — state lives in existing user YAML files as per Typemill architecture.