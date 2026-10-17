# Private security report: Host Header Injection in Typemill password recovery enables reset-token leakage and account takeover

Hi Typemill maintainers,

I found a Host Header Injection / password-reset link poisoning issue in Typemill's password recovery flow. I tested this only on local/lab instances and on an attacker-controlled test subdomain for evidence collection. I did not test any third-party deployments.

If you confirm this as a security vulnerability, I would appreciate reporter credit and would be happy to coordinate CVE assignment and disclosure timing privately.

## Summary

The unauthenticated password recovery endpoint builds the password reset URL using request-derived host metadata. By sending a recovery request with an attacker-controlled `Host` header, an attacker can cause Typemill to email the victim a password reset link pointing to an attacker-controlled domain. If the victim clicks the link, the recovery token is disclosed to the attacker, while the same token is valid on the real Typemill instance. The attacker can then reset the victim's password and log in as that user, including an administrator account.

## Affected version tested

- Repository: `https://github.com/typemill/typemill`
- Branch: `v2.x`
- Commit tested: `31b72ea` (`update composer`)
- Release equivalence in local notes: `2.23.0`
- Tested on: 2026-05-23T14:19:35Z
- Runtime: PHP 8.3.6 CLI with `php -S 127.0.0.1:<port> -t .`

I have not exhaustively tested older versions, but the affected code path appears generic to the current v2.x password recovery implementation.

## Preconditions

- Password recovery is enabled: `recoverpw: true`.
- The attacker knows or guesses the target user's email address.
- The victim clicks the reset link received from the legitimate Typemill instance.
- The deployment accepts/forwards untrusted `Host` values to the application, or a proxy configuration allows attacker-controlled host metadata through.

## Impact

- Unauthenticated attacker can trigger a password-reset email for a victim.
- The reset email contains an attacker-controlled host in the reset URL.
- Victim click leaks `username` and `recovertoken` to the attacker-controlled domain.
- The token is valid server-side for the victim and is not bound to the victim's session/IP.
- The attacker can submit the token to the real Typemill instance, set a new password, and log in as the victim.
- If the target email belongs to an administrator, this leads to administrator account takeover.

## Root cause

The recovery URL is constructed from `$urlinfo['baseurl']`, which is derived from the inbound request URI/host.

Observed code path:

- `system/typemill/system.php`
  - `UriFactory::createFromGlobals($_SERVER)` uses request metadata, including `Host`.
  - `Urlinfo::getUrlInfo($basepath, $uri, $settings)` receives this URI.
- `system/typemill/Static/Urlinfo.php`
  - `baseurl` is composed from the URI scheme and authority.
- `system/typemill/Controllers/ControllerWebRecover.php::recoverPassword()`
  - Builds the reset URL with `$urlinfo['baseurl'] . '/tm/reset?...&recovertoken=...'`.
  - Sends this URL in the password recovery email.
  - Persists the same `recovertoken` server-side for the user.

Because `baseurl` is taken from attacker-controllable host metadata instead of a configured canonical site URL or a strict allowed-host list, the email reset link can be poisoned.

## Safe reproduction steps used in local lab

The following was tested locally with no third-party targets. A local mail-dump instrumentation was used to avoid sending real email and to observe the generated email body.

1. Start Typemill locally:

```bash
cd typemill
php -S 127.0.0.1:9876 -t .
```

2. Configure a local administrator user and enable password recovery:

```yaml
recoverpw: true
mailprotocol: mail
mailfrom: admin@local.test
mailfromname: admin
```

3. Trigger password recovery with an attacker-controlled Host header:

```bash
curl -sS -X POST http://127.0.0.1:9876/tm/recover \
  -H 'Host: hhi-poc.zepatrol.xyz' \
  -H 'Referer: http://127.0.0.1:9876/tm/recover' \
  --data-urlencode 'email=admin@local.test' \
  --data-urlencode 'personal-honey-mail=' \
  -o /tmp/recover.html -w 'HTTP %{http_code}\n'
```

4. Observe the generated email body. It contained a reset URL using the attacker-controlled host:

```text
http://hhi-poc.zepatrol.xyz:9876/tm/reset?username=admin&recovertoken=[TOKEN-REDACTED]
```

5. Confirm server-side state: the same token from the generated email was persisted in `settings/users/admin.yaml` as `recovertoken`.

6. Simulate the victim clicking the link using an attacker-controlled domain (`hhi-poc.zepatrol.xyz`) served through Cloudflare Tunnel to a local capture server. The capture page received:

```text
/tm/reset?username=admin&recovertoken=<same-token>
```

## Evidence collected

Local verification results:

```text
recover_http: 200
mail_contains_domain: YES
token_matches_server_state: YES
token_captured_on_domain: YES
victim_click_http: 200
```

Sanitized generated email evidence:

```text
=== TO: admin@local.test
=== SUBJ: Recover your password
=== BODY:
Dear user,<br/><br/>please use the following link to set a new password:<br/><br/>
<a href="http://hhi-poc.zepatrol.xyz:9876/tm/reset?username=admin&recovertoken=[TOKEN-REDACTED]">
http://hhi-poc.zepatrol.xyz:9876/tm/reset?username=admin&recovertoken=[TOKEN-REDACTED]
</a>
```

A screenshot is attached showing the attacker-controlled capture domain receiving `/tm/reset?username=admin&recovertoken=...` and showing the sanitized generated email evidence.

## Severity / CVSS proposal

Suggested severity: High / Critical depending on scoring policy.

Conservative CVSS 3.1 proposal:

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H
Score: 8.1 High
```

Rationale:

- Network exploitable.
- Low complexity.
- No privileges required to trigger the recovery email.
- Requires user interaction: victim clicks the poisoned link in the legitimate recovery email.
- Successful exploitation can compromise confidentiality, integrity, and availability of the victim account; administrator targeting can compromise the whole Typemill instance.

## CWE mapping

Possible CWE categories:

- CWE-601: URL Redirection to Untrusted Site ('Open Redirect') — reset link poisoning aspect.
- CWE-20: Improper Input Validation — untrusted Host header accepted for security-sensitive URL generation.
- CWE-640: Weak Password Recovery Mechanism for Forgotten Password — password reset token disclosure through poisoned recovery link.

## Recommended fix

1. Do not build security-sensitive links from request `Host` / request authority.
2. Add a configured canonical public base URL, e.g. `settings['baseurl']`, and use it for password reset, account activation, and any other security-sensitive absolute links.
3. If no canonical base URL is configured, fail closed for password recovery instead of sending a link with a request-derived host.
4. Add strict allowed-host validation at application entry, rejecting unexpected Host values early.
5. Only trust `X-Forwarded-Host` / `Forwarded` values from known reverse proxies, and document secure proxy configuration.
6. Add regression tests ensuring password recovery emails do not reflect malicious `Host` or `X-Forwarded-Host` values.

Example pattern:

```php
$canonicalBase = $settings['baseurl'] ?? null;
if (!$canonicalBase) {
    // Fail closed: do not send a reset link with an attacker-controllable host.
    return $response
        ->withHeader('Location', $this->routeParser->urlFor('auth.recoverform'))
        ->withStatus(302);
}

$url = rtrim($canonicalBase, '/')
     . '/tm/reset?username=' . urlencode($requiredUser['username'])
     . '&recovertoken=' . $recovertoken;
```

## Cleanup / safety notes

- The test was performed on a local Typemill instance and an attacker-controlled test subdomain.
- No third-party Typemill deployments were tested.
- No real user email account was used.
- The temporary capture server was stopped after evidence collection.
- The temporary mail instrumentation was removed after testing.

## Fix applied

### Solution

A minimal fix was applied to `system/typemill/Controllers/ControllerWebRecover.php` in the `recoverPassword()` method. The password reset URL is now built from a trusted source instead of the attacker-controllable `urlinfo['baseurl']`.

**Code change:**

```php
// Before:
$url = $urlinfo['baseurl'] . '/tm/reset?username=' . $requiredUser['username'] . '&recovertoken=' . $recovertoken;

// After:
$baseUrl = !empty($settings['fqdn']) ? rtrim($settings['fqdn'], '/') : ($urlinfo['scheme'] . '://' . ($_SERVER['SERVER_NAME'] ?? $urlinfo['authority']) . $urlinfo['basepath']);
$url = rtrim($baseUrl, '/') . '/tm/reset?username=' . $requiredUser['username'] . '&recovertoken=' . $recovertoken;
```

**How it works:**

1. **If `fqdn` is configured** (Developer → Proxy settings) → use that explicit canonical URL.
2. **If `fqdn` is empty** → fall back to `$_SERVER['SERVER_NAME']`, which is the web server’s configured hostname (nginx `server_name` / Apache `ServerName`). This is **not** derived from the client’s `Host` header.
3. **If `SERVER_NAME` is unavailable** → final fallback to the request-derived authority.

**Why this is safe:**
- No stored values are auto-captured or persisted during setup.
- Moving the site from localhost to a live server automatically picks up the new `SERVER_NAME` at runtime.
- Zero side effects on the rest of the application (only the password recovery email URL changes).
- No new UI fields or settings schema changes.

### Limitations & next steps

- **Apache `UseCanonicalName Off` (default):** On Apache with the default `UseCanonicalName Off`, `SERVER_NAME` equals the request `Host` header, so the fallback is still vulnerable on that specific configuration. Production Apache reverse-proxy setups usually set `UseCanonicalName On` or configure `ServerName`, which makes this safe.
- **Recommended follow-up:** Add a new dedicated `mailbaseurl` setting in the system settings UI, explicitly for email links. This gives the admin full control without affecting the rest of the site.
- **Defense in depth:** Consider binding the recovery token to a server-side secret (e.g., hash the token with `SERVER_NAME` before storing). This makes captured tokens useless even if the URL is somehow poisoned.
- **Regression test:** Add a test that sends a password recovery request with a malicious `Host` header and asserts the email body does not contain the attacker-controlled host.

Thank you for reviewing this report.
