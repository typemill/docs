# Setup the Password Recovery

Typemill ships with a built-in password recovery. You can activate and configure the password recovery feature in the system settings. But before you setup a password recovery, please take some time to think about passwords again.

![Screenshot of the password recovery of Typemill](media/live/password-recovery.webp){.center loading="lazy" width="820" height="442"}

## Do You Really Need a Password Recovery?

We made the password recovery feature as secure as possible. However, a password recovery function always has security implications. We recommend using this feature only if you really need it (e.g., many different users and authors). As an admin, you can always reset the password for every user, so for small teams, a password recovery is usually not needed.

## Lost Your Admin Password?

If you are the webmaster and have lost your admin password without having activated the password recovery, then you can recover your account with the following steps:

* Use another installation of Typemill (e.g., a local installation or test installation).
* Create a new admin user.
* Connect to your website (e.g., via FTP).
* Delete the old admin user and upload the new admin user in the folder settings > users.

## Use Secure Passwords

The most important step is to use secure passwords. We highly recommend using a password manager like KeePass or others to manage all your passwords so you will never lose a password again. As a first step, you can also use the password manager of your browser.

Some general advice:

* Use a password manager like KeePass to handle all your passwords.
* Use strong and unique passwords for all applications; never use the same password for different applications. A password should have at least 8 characters and contain all kinds of signs like uppercase and lowercase letters, numbers, and special characters.
* Passwords should be random and not guessable. If you really want to create a password yourself to remember, then you can create a short sentence, add a lot of lexical errors, mix uppercase and lowercase, and add random numbers and special characters into it.

## Setup an Email for Notifications

Before you configure and activate the password recovery, make sure that you enter a valid email address in the email section of the system settings and that you send a test email to ensure that your email account accepts the recovery emails. Some email providers like Google Mail might reject emails from this feature.

## How the Password Recovery Works

The recovery feature works as follows:

* In the frontend, the user will see a link below the login form that leads to the recovery form.
* In the recovery form, the user can add his email.
* The system will send an email with a recovery link if it finds the email in the user database.
* The link will be valid for 1 day.
* The link will open a form where the user can enter a new password.

