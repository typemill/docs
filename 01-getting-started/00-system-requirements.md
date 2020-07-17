# System Requirements

Typemill is a modern and lightweight software with nearly no requirements. All you need is:

- **PHP 7.0+**
- **A webserver (Apache)**
- **mod_rewrite** and **htaccess**

!! **For Linux users**
!! 
!! If you run a linux system, then please double check that `mod_rewrite` and `htaccess` are active.

Almost any hosting package provides a webserver with PHP. If you've ever hosted your own website, then chances are high that you can run TYPEMILL there without any problems.

!!**Build-in PHP**
!!
!!If you are using the build-in php-server on Windows or Mac, then make sure that some basic libraries are installed. Typemill needs `mbstring`, `iconv`,  `session`, `gd`, and `fileinfo`. You can check that with `phpinfo()`. 

You will need FTP software to upload TYPEMILL to your webserver.

Typemill supports the following browsers:

- Firefox (heavily tested)
- Chrome (tested)
- Edge (basic tests)
- IE11 (basic tests)
- Safari (not tested)

I use Firefox and Linux for development. If you have problems with other environments, then please let me know, and add a new issue on [GitHub](https://github.com/typemill/typemill).

