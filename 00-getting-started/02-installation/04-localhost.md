#  Run Typemill on Localhost

You can set up and test Typemill on your local machine. With the domain `localhost` and subdomains `typemilltest.`, you can also test all premium plugins and premium themes without a license and without any restrictions.

To run Typemill on your local machine, simply download Typemill (zip or git) and place the files in a folder like `localhost/typemill`. Check the requirements, ensure PHP and a web server like Apache are installed, and verify the permissions for the files and folders. If everything is configured correctly, then no additional work is required.

! **Move to Production**
! 
! You can create a complete web project on your local machine and simply move it to the production server when it is ready to launch. After moving the website to another domain, you have to clear the cache and generate a fresh navigation with the new base URL. Just open the Kixote interface and type `clear cache`, and after that, `clear navigation`, and you are done.

If you have never worked with a PHP setup on localhost, then read the information for Linux, Windows, and Mac below. If you encounter any trouble, please consult the troubleshooting guide.

## Run on Linux

If you want to run Typemill on your local Linux machine, then you only need:

* A PHP installation.
* A web server like Apache or Nginx.

We cannot provide a full guide to set up your environment, but in most cases, setting up PHP is as simple as this:

```
sudo apt install php
```

After that, check the PHP version with:

```
sudo php --version
```

Make sure you have installed all required PHP libraries with:

```
sudo apt install php-gd php-mbstring php-fileinfo php-iconv php-session php-openssl php-curl
```

Depending on your server, you may need to restart the service. For Apache, type:

```
sudo systemctl restart apache2
```

Ensure that the Apache server has access to all files:

```
sudo chown -R www-data:www-data /var/www/typemill
```

Make sure that the folders `cache`, `content`, `data`, `media`, and `settings` are writable. It is recommended to set folder permissions to 755 and file permissions to 644.

For more details, please refer to the guides for  Apache an Nginx.

## Run on Windows

For Windows machines, we recommend using [xampp](https://www.apachefriends.org/de/download.html) or [Laragon](https://laragon.org/download/) to set up a local environment with PHP and Apache. You can also use Docker if you are familiar with that tool.

## Run on Mac

Install PHP on your Mac with:

```
brew install php
```

Check the PHP version with:

```
php -v
```

Then open the config file of the built-in Apache server with:

```
sudo nano /etc/apache2/httpd.conf
```

Uncomment the following module:

```
LoadModule php_module /opt/homebrew/opt/php/lib/httpd/modules/libphp.so
```

Start the built-in Apache server:

```
sudo apachectl start
```

