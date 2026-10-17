# Plan: Integrate PHPMailer into Typemill Core

## Goal

Replace the native-`mail()`-only `SimpleMail` model with a PHPMailer-based `Mail` model in Typemill core. This enables SMTP support for core features like 2FA (authcode) and password recovery, making them work out-of-the-box in Docker environments without requiring the mail plugin.

---

## Current State

### `SimpleMail` (core)

- File: `system/typemill/Models/SimpleMail.php`
- Uses PHP native `mail()` only — no SMTP, no Docker compatibility.
- Reads settings keys: `mailfrom`, `mailfromname`, `mailreply`.
- **Bug:** The form field is named `replyto` but `SimpleMail` reads `mailreply` — Reply-To is effectively non-functional.
- **Bug:** Malformed Reply-To header: `'Reply-To: base64' . $this->reply` (the literal string `"base64"` is prepended to the address).

### Mail Plugin (`plugins/mail/`)

- Has PHPMailer v6.8.1 vendored locally in `plugins/mail/vendor/`.
- Subscribes to the event `onTwigLoaded` — **this event is never dispatched** by `system.php`. The plugin's DI container registration (`$container->set('mail', ...)`) therefore never executes.
- The plugin is effectively broken for its intended purpose.
- Additional bugs in `MailHandler.php`: `addCC()` uses undefined `$address` variable; `send()` catches `phpmailerException` (unqualified, wrong class) instead of `\PHPMailer\PHPMailer\Exception`.

### Controllers using `SimpleMail`

| Controller | Usage |
|---|---|
| `Controllers/ControllerWebAuth.php` | 2FA/authcode delivery during login |
| `Controllers/ControllerWebRecover.php` | Password recovery email with reset link |
| `Controllers/ControllerApiTestmail.php` | Admin test-mail action |

### PHPMailer availability

- **Not** in root `composer.json` or `system/vendor/`.
- Present only in `plugins/mail/vendor/phpmailer/phpmailer/` (v6.8.1).

### Secrets / password storage mechanism

`Settings::findSecurityDefinitions()` scans YAML form definitions for fields typed `password`. `ControllerApiSystemSettings` calls this automatically, strips those values from the regular settings save, and writes them to `settings/secrets.yaml` via `Settings::updateSecuritySettings()`. Reading back uses `Settings::getSecret($fieldname)`. This mechanism already handles `chatgptKey` and `claudeKey` in `system.yaml` — no new infrastructure is needed for an SMTP password field.

---

## Decisions

| Decision | Choice | Reason |
|---|---|---|
| Mail plugin fate | **Remove entirely** | The plugin's DI registration never executed (broken event), its functionality moves to core, and keeping it would create conflicting UI. |
| SMTP password storage | **`secrets.yaml` via existing mechanism** | Consistent with how other API keys (ChatGPT, Claude) are handled; zero new infrastructure. |

---

## Step-by-Step Implementation Plan

### Step 1 — Add PHPMailer to Root `composer.json`

**File:** `/var/www/html/typemill/composer.json`

Add to the `require` block:

```json
"phpmailer/phpmailer": "^6.0"
```

Then run `composer update`. PHPMailer installs into `system/vendor/phpmailer/phpmailer/` (the project's vendor dir is already `system/vendor` per `composer.json` config).

---

### Step 2 — Extend `system.yaml` with SMTP Form Fields

**File:** `system/typemill/settings/system.yaml`  
**Location:** Inside the `fieldsetmail` fieldset, after the existing `replyto` field (line 281).

Add the following fields:

```yaml
    mailsmtp:
      type: checkbox
      label: 'Use SMTP'
      checkboxlabel: 'Send mails via SMTP instead of PHP mail()'
      description: 'Activate SMTP to send mails via an external mail provider. Required for Docker and most hosting environments.'
    mailhost:
      type: text
      label: 'SMTP Host'
      placeholder: 'smtp.gmail.com'
      maxlength: 100
    mailport:
      type: text
      label: 'SMTP Port'
      placeholder: '587'
      maxlength: 10
    mailsmtpsecure:
      type: select
      label: 'SMTP Security'
      options:
        '': None
        ssl: SSL/TLS
        tls: TLS/STARTTLS
    mailsmtpauth:
      type: checkbox
      label: 'SMTP Authentication'
      checkboxlabel: 'Authenticate via SMTP'
    mailusername:
      type: email
      label: 'SMTP Username'
      placeholder: 'your@email.com'
      autocomplete: off
      maxlength: 100
    mailpassword:
      type: password
      label: 'SMTP Password'
      autocomplete: new-password
      description: 'For security reasons, your password is stored separately and will not be visible again after you leave this page.'
```

The `mailpassword` field with `type: password` is automatically routed to `secrets.yaml` by the existing `ControllerApiSystemSettings` → `Settings::findSecurityDefinitions()` → `extractSecuritySettings()` → `updateSecuritySettings()` flow. **No controller changes are needed for this.**

---

### Step 3 — Create `Typemill\Models\Mail.php`

**File:** `system/typemill/Models/Mail.php` (new file, replaces `SimpleMail.php`)

Full specification:

```php
namespace Typemill\Models;

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;
use Typemill\Static\Translations;

class Mail
{
    public $error;

    private string $from      = '';
    private string $fromName  = '';
    private string $replyTo   = '';
    private array  $settings;

    public function __construct(array $settings)
    {
        // reads: mailfrom, mailfromname, replyto (fix: was 'mailreply' in SimpleMail — mismatch with form field)
        // reads SMTP config: mailsmtp, mailhost, mailport, mailsmtpsecure, mailsmtpauth, mailusername
        // reads SMTP password via Settings::getSecret('mailpassword')
    }

    public function send(string $to, string $subject, string $message): bool
    {
        // 1. Validate $this->from is set, else set $this->error and return false
        // 2. Instantiate PHPMailer
        // 3. If mailsmtp: configure SMTP (host, port, secure, auth, username, password)
        //    else: $mail->isMail()  ← same behavior as current SimpleMail fallback
        // 4. Set From, FromName, Sender, optional ReplyTo
        // 5. Set charset UTF-8
        // 6. addAddress($to)
        // 7. isHTML(true), Subject, Body (HTML), AltBody (strip_tags fallback)
        // 8. send() — on PHPMailer exception: set $this->error, return false
        // 9. return true on success
    }
}
```

**Bugs fixed compared to `SimpleMail`:**
- Settings key `mailreply` → correct key `replyto` (matches form field in `system.yaml`)
- Malformed Reply-To header `'Reply-To: base64' . $reply` → proper `$mail->addReplyTo()`
- Uses PHPMailer from `system/vendor` (not plugin-local vendor)

---

### Step 4 — Update Three Controllers (Minimal Change)

Each controller needs exactly **2 lines changed**: the `use` import and the constructor call.

**`Controllers/ControllerWebAuth.php`** (line 9, line 627):
```php
// Before:
use Typemill\Models\SimpleMail;
$mail = new SimpleMail($this->settings);

// After:
use Typemill\Models\Mail;
$mail = new Mail($this->settings);
```

**`Controllers/ControllerWebRecover.php`** (line 10, line 54):
```php
// Same swap
```

**`Controllers/ControllerApiTestmail.php`** (line 9, line 39):
```php
// Same swap
// Note: the pre-check on line 16 for $this->settings['mailfrom'] stays unchanged
```

No logic changes to any controller — the `send(string $to, string $subject, string $message): bool` and `$error` property API are preserved identically.

---

### Step 5 — Delete `SimpleMail.php`

**File:** `system/typemill/Models/SimpleMail.php` — delete after all controller references are updated.

---

### Step 6 — Register `Mail` in the DI Container (Recommended)

**File:** `system/typemill/system.php`  
**Location:** After `$container->set('settings', ...)` (around line 239).

```php
$container->set('mail', function() use ($settings) {
    return new \Typemill\Models\Mail($settings);
});
```

This allows plugins to resolve `$this->container->get('mail')` to send mail through the same configured service. Follows the existing DI pattern used by settings, ACL, dispatcher, etc.

---

### Step 7 — Delete the Mail Plugin

**Directory:** `plugins/mail/` — remove entirely.

**Why no regression:**
- The plugin subscribed to `onTwigLoaded`, which is never dispatched by `system.php`. Its `$container->set('mail', ...)` closure therefore never ran in any Typemill installation.
- Any install with the plugin listed as "active" in `settings/settings.yaml` will simply have a stale `plugins.mail.active: true` entry. On next settings save, `system.php` re-scans the plugins directory, detects the count mismatch, and calls `$settingsModel->updateSettings($pluginSettings, 'plugins')` to clean up the orphaned entry automatically (lines 172–227 of `system.php`).

---

## Files Changed Summary

| Action | File |
|---|---|
| Edit | `composer.json` |
| Edit | `system/typemill/settings/system.yaml` |
| **New** | `system/typemill/Models/Mail.php` |
| **Delete** | `system/typemill/Models/SimpleMail.php` |
| Edit (2 lines) | `system/typemill/Controllers/ControllerWebAuth.php` |
| Edit (2 lines) | `system/typemill/Controllers/ControllerWebRecover.php` |
| Edit (2 lines) | `system/typemill/Controllers/ControllerApiTestmail.php` |
| Edit | `system/typemill/system.php` (DI registration, optional but recommended) |
| **Delete** | `plugins/mail/` (entire directory) |

---

## Bugs Fixed Along the Way

| Bug | Location | Fix |
|---|---|---|
| `replyto` form field never matched `mailreply` settings key | `SimpleMail.php` | Use `replyto` consistently in `Mail.php` |
| Malformed Reply-To header (`'Reply-To: base64' . $reply`) | `SimpleMail.php` | Use `$mail->addReplyTo()` in PHPMailer |
| `addCC()` uses undefined `$address` variable | `MailHandler.php` | N/A — `MailHandler` is deleted with the plugin |
| Wrong exception class `phpmailerException` in `send()` | `MailHandler.php` | N/A — deleted with plugin |

---

## Out of Scope

- No database changes.
- No JWT.
- No changes to the Slim 4 / PHP-DI architecture.
- No changes to frontend Vue components (the existing mail settings form in admin already renders `fieldsetmail` dynamically from YAML).
- No migration script needed — SMTP settings are purely additive; existing installs without SMTP configured continue to use `isMail()` fallback as before.

---

## Execution Report

All 9 steps were executed and verified. No deviations from the plan.

---

### Step 1 — `composer.json` updated, PHPMailer installed

**Result:** PHPMailer **v6.12.0** (latest stable, up from v6.8.1 in the old plugin) was installed into `system/vendor/phpmailer/phpmailer/`.

Change applied to `composer.json`:
```json
"phpmailer/phpmailer": "^6.0"
```

Verified at: `system/vendor/phpmailer/phpmailer/src/PHPMailer.php` — present and autoloadable.

---

### Step 2 — `system.yaml` extended with SMTP fields

**Result:** 7 fields appended inside `fieldsetmail`, after the existing `replyto` field (now at lines 282–318).

Fields added exactly as planned:
- `mailsmtp` (checkbox)
- `mailhost` (text)
- `mailport` (text)
- `mailsmtpsecure` (select: None / SSL/TLS / TLS/STARTTLS)
- `mailsmtpauth` (checkbox)
- `mailusername` (email)
- `mailpassword` (password → auto-routed to `secrets.yaml`)

---

### Step 3 — `Mail.php` created

**Result:** `system/typemill/Models/Mail.php` created, 113 lines. Full implementation (not pseudocode).

Key implementation details:
- `PHPMailer(true)` — exceptions enabled
- Charset set via `PHPMailer::CHARSET_UTF8` constant
- SMTP path: configures host, port (cast to `int`), SMTPSecure, SMTPAuth, Username; reads password via `Settings::getSecret('mailpassword')`
- `isMail()` fallback when `mailsmtp` is not set or false
- `AltBody` set via `strip_tags($message)` for plain-text clients
- On exception: `$this->error = $mail->ErrorInfo` (PHPMailer's full error detail, better than the old `error_get_last()`)

---

### Step 4 — Three controllers updated

**Result:** All three controllers updated with exact 2-line swap as planned. Verified line numbers:

| File | `use` line | instantiation line |
|---|---|---|
| `ControllerWebAuth.php` | 10 | 627 |
| `ControllerWebRecover.php` | 10 | 54 |
| `ControllerApiTestmail.php` | 9 | 39 |

No other logic changed in any controller.

---

### Step 5 — `SimpleMail.php` deleted

**Result:** `system/typemill/Models/SimpleMail.php` removed. Confirmed absent. No remaining functional references to `SimpleMail` anywhere in the codebase (one comment in `Mail.php` notes the historical bug fix).

---

### Step 6 — DI container registration added

**Result:** Added to `system/typemill/system.php` at line 242, immediately after `$container->set('settings', ...)`:

```php
# add mail service to the container so plugins can send mail via $this->container->get('mail')
$container->set('mail', function() use ($settings){ return new \Typemill\Models\Mail($settings); });
```

---

### Step 7 (plan Step 7) — `plugins/mail/` deleted

**Result:** Entire `plugins/mail/` directory removed including its locally vendored PHPMailer copy, `MailHandler.php`, `mail.php`, `mail.yaml`, and all templates.

---

### Verification Checks

| Check | Result |
|---|---|
| `plugins/mail/` exists | No — correctly deleted |
| `Models/SimpleMail.php` exists | No — correctly deleted |
| `system/vendor/phpmailer/phpmailer/src/PHPMailer.php` exists | Yes |
| `composer.json` contains `phpmailer/phpmailer` | Yes — `"^6.0"` |
| `composer.lock` resolved version | v6.12.0 |
| `ControllerWebAuth.php` uses `Mail` | Yes — lines 10, 627 |
| `ControllerWebRecover.php` uses `Mail` | Yes — lines 10, 54 |
| `ControllerApiTestmail.php` uses `Mail` | Yes — lines 9, 39 |
| `system.php` registers `mail` in DI container | Yes — line 242 |
| `Mail` class instantiates via autoloader | Verified with `php -r` — `Typemill\Models\Mail instantiated OK` |
