#  Run Typemill with Proxy Detection

If you want to run Typemill with a proxy, activate the proxy detection in the developer settings. To improve security, you can also add several trusted IPs, separated by commas.

! **Delete Cache**
! 
! Remember that Typemill uses a cached version of the navigation that contains absolute URLs. If you switch to a proxy setup, you must delete the cached navigation (folder -> data -> navigation). On the next visit, Typemill will regenerate the cached navigation with the proxy URLs.

The following test setup on localhost with an Apache server uses the URL `https://localhost` to access a Typemill installation in a subfolder under `http://localhost/typemillProxy`.

```
<VirtualHost *:443>
    ServerName localhost

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

    # Enable rewrite engine
    RewriteEngine On

    # Rewrite all requests to include /typemillProxy/ internally
    RewriteRule ^/(.*)$ /typemillProxy/$1 [PT]

    RequestHeader set X-Forwarded-Proto "https"

    ProxyRequests Off
    ProxyPreserveHost On

    # ProxyPass configurations
    ProxyPass /typemillProxy/ http://localhost/typemillProxy/
    ProxyPassReverse /typemillProxy/ http://localhost/typemillProxy/

    <Directory "/var/www/html/typemillProxy">
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

