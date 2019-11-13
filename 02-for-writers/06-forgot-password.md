# Lost Your Password?

TYPEMILL does not provide password recovery, but there are two ways to create a new password:

## Is there another admin?

If there is another user with admin rights, then contact them. They can delete your user account, create a new one, and tell you the new password. Don't forget to change the password again.

## No other admin?

If you are the only admin user, then please follow these steps: 

* Connect to your website (e.g. via FTP).
* Go to the folder `/settings` and backup the file `settings.yaml`.
* Then delete the file `settings.yaml` on your server.
* Go to `yoursite.com/setup`.
* Fill out the form. This will create a new admin user and a fresh settings-file.
* Upload your old settings-file, so your old settings are active again.
* If you haven't done this already, delete the old admin user in the user management screen now.

It might seem a bit inconvenient, but it makes sure that you are the owner of the website.

