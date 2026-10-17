

Hey team, I found Username Uniqueness Bypass via Newline Character Injection (\n) in your software so I am requesting you to fix this issue & assign a CVE for tracking purposes on github security adversaries.

Title: Username Uniqueness Bypass via Newline Character Injection (\n)

Severity: Medium–High

Description: The registration functionality does not sanitize control characters in the username field. This allows an attacker to register a duplicate username by appending a newline character (\n), bypassing the uniqueness check and creating accounts that are visually identical to existing ones.

Steps to Reproduce:

    Register an account with username admin
    Register a second account using admin\n (with a newline character) in the username field
    Observe that registration succeeds and both accounts appear as admin in the UI

Impact:

    Account confusion in logs, audit trails, and user management panels
    An attacker can impersonate existing users or administrators
    Potential privilege escalation if role logic matches on username
    Further bypass possible with similar characters (\r, \t, \0, unicode lookalikes)

Root Cause: The application does not strip or normalize control characters before performing the username uniqueness check, and appears to lack a database-level unique constraint that would catch this.
Example Suggested Fix:  username = username.trim().replace(/[\r\n\t\0]/g, '');  &  Additionally, enforce a unique constraint at the database level as a secondary safeguard.

Fix Applied (2026-06-10):

    Added a sanitizeUsername() method in system/typemill/Models/User.php that strips \r, \n, \t, \0 and trims whitespace.
    Applied sanitization to all filesystem-bound user operations:
        setUser(), setUserWithPassword(), createUser(), updateUser(), deleteUser()
    Hardened validation rules in system/typemill/Models/Validation.php:
        userAvailable and userExists now sanitize the input before checking uniqueness/existence.
    Root cause of bypass: PHP PCRE's $ anchor matches before a trailing newline, so Valitron's alphaNum rule incorrectly allowed "admin\n" to pass. The uniqueness check then looked for admin\n.yaml (non-existent) instead of admin.yaml, allowing the duplicate.
    Side effects: None for legitimate users. Only control characters that were already invalid by the alphaNum rule are removed.

References: 