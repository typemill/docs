#  PHP Versions and Extensions

Typemill currently requires the following PHP versions:

* PHP 8.1
* PHP 8.2
* PHP 8.3
* PHP 8.4 supported by Typemill Core but not by all plugins.

Typemill uses some standard extensions that are supported by most hosting providers out of the box. If you use minimal PHP distributions like PHP-FPM, ensure that the following extensions are installed:

* php-fileinfo
* php-gd
* php-iconv
* php-mbstring
* php-session

The following extensions are highly recommended, but there are fallbacks if they are missing:

* php-openssl
* php-curl

Some plugins might require additional extensions, such as:

* php-xml (ebook plugin)
* php-zip (ebook plugin)

You can check for these libraries with `phpinfo()`.

