#  Run Typemill on Apache Servers

Please check if your Apache server meets the [system requirements](https://typemill.net/tm/content/visual/getting-started/system-requirements). In 99% of the cases, you will only have to configure your `.htaccess` file to start with Typemill.

## .htaccess 

Typemill ships with a `.htaccess` file and requires `mod_rewrite` on Apache servers. Ensure your host supports `mod_rewrite`. If you run Typemill locally, you can enable `mod_rewrite` in Linux with:

```bash
$ sudo a2enmod rewrite 
```

In some environments (like 1und1), you have to use an empty slash as the rewrite base. If you have any trouble (e.g., getting a 500 error from the server), then please activate this line in your `.htaccess` and try again:

```
RewriteBase /
```

You can configure several details in the `.htaccess` file, such as HTTPS, slashes, www, and subfolders. Simply uncomment the lines if needed:

```
# If your homepage is http://yourdomain.com/yoursite
# Set the RewriteBase to:
# RewriteBase /yoursite

# In some environments, an empty RewriteBase is required:
# RewriteBase /

# Use this to redirect HTTP to HTTPS on Apache servers
# RewriteCond %{HTTPS} off
# RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

# Use this to redirect www to non-www on Apache servers
# RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
# RewriteRule ^(.*)$ http://%1/$1 [R=301,L]

# Use this to redirect slash/ to URL without slash on Apache servers
# RewriteCond %{REQUEST_FILENAME} !-d
# RewriteRule ^(.*)/$ /$1 [R=301,L]

```

## Configure Apache on Linux/Ubuntu

Make sure that you have Apache2 and PHP installed:

```bash
sudo apt update
sudo apt install apache2
sudo apt install php libapache2-mod-php
```

Open your Apache configuration file:

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Ensure the `AllowOverride` directive is set:

```apache
<Directory /var/www/html>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

