#  Email Tab

In the email tab, you can define an email address for sending notifications such as password recovery and login verification emails. Please test the email feature using the test button before activating features that rely on emails.

![Screenshot of the email tab of Typemill](media/live/email-tab.webp){.center loading="lazy" width="787" height="498"}

## Options

| Option                  | Description                                                                                                                      |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| Mail from (required)    | Enter an email address that will send the emails (the sender). This field is prefilled with the email you entered during setup. Send a test email to your user account to verify that you receive the emails. |
| Mail from name (optional)| Optionally, enter a name for the sender address. If not set, the sender's email address will be visible.                       |
| Reply to (optional)     | Optionally, enter a reply-to address for responses from the receiver. If not set, responses will go to the sender's address.   |

## Important Notes

The `mail()` function relies on an SMTP server to send emails. Many shared hosting providers disable `mail()` for security reasons or require additional configuration. Some hosts allow `mail()` but require authenticated SMTP settings (e.g., via `php.ini` or PHPMailer).

To prevent emails from being marked as spam, your domain should have the correct DNS records for SPF, DKIM, and DMARC:

* **SPF** (Sender Policy Framework): Defines which mail servers are allowed to send emails for your domain. 
* **DKIM** (DomainKeys Identified Mail): Signs outgoing emails to verify their legitimacy.
* **DMARC** (Domain-based Message Authentication): Reporting & Conformance: Instructs receiving mail servers on handling SPF/DKIM failures.

Please consult your hosting provider for more details.

