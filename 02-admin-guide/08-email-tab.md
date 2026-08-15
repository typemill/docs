#  Email Tab

In the email tab, you can define the sender address for notification emails such as password recovery and login verification. You can also activate SMTP to send mails via an external mail provider instead of PHP `mail()`. This is required for Docker and most hosting environments.

Please test the email feature using the test button before activating features that rely on emails.

![Screenshot of the email tab of Typemill](media/live/email-tab.webp){.center loading="lazy" width="787" height="498"}

## Options

| Option                    | Description                                                                                                                      |
|---------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| Mail from (required)      | Enter the email address that will send the emails (the sender). This field is prefilled with the email you entered during setup. Send a test email to your user account to verify that you receive the emails. |
| Mail from name (optional)  | Optionally, enter a name for the sender address. If not set, the sender's email address will be visible.                       |
| Reply to (optional)        | Optionally, enter a reply-to address for responses from the receiver. If not set, responses will go to the sender's address.   |
| Use SMTP                  | Activate SMTP to send mails via an external mail provider instead of PHP `mail()`.                                              |
| SMTP Host                  | Enter the hostname of your SMTP server.                                                                                          |
| SMTP Port                  | Enter the port used by your SMTP server.                                                                                          |
| SMTP Security              | Select the encryption method used for the SMTP connection.                                                                      |
| SMTP Authentication        | Enable SMTP authentication if your mail provider requires login credentials.                                                     |
| SMTP Username              | Enter the username for SMTP authentication.                                                                                       |
| SMTP Password              | Enter the password for SMTP authentication. For security reasons, your password is stored separately and will not be visible again after you leave this page. |

## Important Notes

Using SMTP is recommended for most hosting environments, especially Docker setups, because PHP `mail()` is often unavailable or restricted.

To prevent emails from being marked as spam, your domain should have the correct DNS records for SPF, DKIM, and DMARC:

* **SPF** (Sender Policy Framework): Defines which mail servers are allowed to send emails for your domain.
* **DKIM** (DomainKeys Identified Mail): Signs outgoing emails to verify their legitimacy.
* **DMARC** (Domain-based Message Authentication, Reporting & Conformance): Instructs receiving mail servers on handling SPF/DKIM failures.

Please consult your hosting provider or mail provider for the correct SMTP settings.

