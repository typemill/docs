# Login Verification

The login verification provides a simple and effective method to enhance the security of your Typemill installation and protect against password theft. Activating this feature is straightforward and can be done in just two steps.

![Screenshot of the login verification screen of Typemill](media/live/login-verification.webp){.center loading="lazy" width="820" height="528"}

## Setup an Email for Notifications

Before activating the login verification, it's crucial to ensure that you have entered a valid email address in the email section of the system settings. Additionally, send a test email to confirm that your email account can receive verification emails successfully. Be aware that some email providers, such as Google Mail, might reject emails from this feature, so it's essential to verify deliverability.

![Screenshot of the email settings in Typemill](media/live/email-tab.webp){.center loading="lazy" width="820" height="519"}

## Activate the login verification

To activate login verification, navigate to the "Security" tab in the system settings.

![screenshot of the security-tab](media/live/security-tab.webp){.center loading="lazy" width="820" height="513"}

## How it Works

* **After a login attempt**, Typemill will display a form for you to enter an authentication code to complete your login.
* **Following a successful login attempt** (with the correct username and password), Typemill will send a 5-digit authentication code to your registered email address.
* Upon **entering the valid authentication code** into the form, your login will be completed.
* You can log in without requiring a new verification code for **24 hours**.

Some more background information:

- If you do not receive the email with the verification code, the login attempt might have used an incorrect password or username. In the worst case, if the credentials were correct but the email was rejected by your email provider, see the troubleshooting chapter below.
- The authentication code is valid for 5 minutes. If you do not complete your login within this time, you will need to start over to receive a new authentication code.
- A successful verification is valid for 24 hours, after which the system will require a new verification code.
- The verification code is only valid for one device. Logging in from another device or browser will necessitate a new verification code.

## Privacy Implications

Typemill stores a simple fingerprint in the user account for each device used for login. These fingerprints are stored as MD5 hashes and include the following information:

* `HTTP_USER_AGENT`
* `REMOTE_ADDR`
* `HTTP_ACCEPT_LANGUAGE`

Ensure this practice complies with the privacy legislation in your country, considering user consent and transparency.

## Troubleshooting

If you encounter any issues, you may need FTP access to deactivate the verification feature manually:

1. Connect to your website via FTP.
2. Download the `settings.yaml` file from the "settings" folder.
3. Change the line `authcode: true` to `authcode: false`.
4. Upload the modified `settings.yaml` back to your website.

## If You Receive an Email Without a Login Attempt

Receiving an email notification without attempting to log in could indicate that your account has been compromised. Immediate actions include:

- **Reset your password immediately.**
- **Inspect for unauthorized changes** to your site. Consider a fresh setup of your Typemill installation and check for any malicious code or files in your content, media, or settings files.
- **Investigate the breach** to understand how your password was compromised and take steps to prevent future incidents.

Implementing login verification is a step forward in securing your Typemill site. Stay vigilant and proactive in enhancing your site's security.

