# Security Tab

In the security tab, you can activate the verification feature, configure a captcha feature, and activate a security log. Please read the article about the [login verification](/admin-guide/login-verification) for all details.

![Screenshot of the security tab of Typemill](media/live/security-tab.webp){.center loading="lazy" width="819" height="513"}

## Options

| Feature               | Description                                                                                         |
|-----------------------|-----------------------------------------------------------------------------------------------------|
| Login verification     | Verify your login with a 5-digit code sent by email. Please read the important notes below.       |
| Use captcha            | Activate a captcha in authentication forms with options: "Disable", "Always show", "Show after first wrong input". |
| Security log           | Track spam and suspicious actions in a log file.                                                  |

## Login Verification

An email is required for this feature. Send a test email before you use this feature. Make sure you have FTP access to disable the feature in settings.yaml in case of failure. The verification code will be valid for 5 minutes. Be aware that device fingerprints will be stored in the user accounts.

## Security Log

Please note that the security log tracks IP addresses for security-related actions like wrong login attempts. Before you activate it, make sure it complies with the legislation in your country.

