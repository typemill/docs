#  Run Typemill with Nginx

This guide has been written by Ezequiel Bruni; you can find the original version on [GitHub](https://gist.github.com/EzequielBruni/05605ca206db6842ce00dc9a45b24194).

## The `.conf` file

Here is the basic file without SSL enabled. This configuration should also work behind a simple Nginx reverse proxy and will work with separate LXD containers (i.e., one container is hosting the website, and another is hosting the proxy server).

Just change everything in \[brackets\] to meet your needs and anything else you feel like changing.

```
server {
    listen 80;
    listen [::]:80; 

    # Your domain name
    server_name [your-domain-name];

    # Document root
    root        [your-root-directory-here];

    # Just setting up some log files
    access_log  /var/log/nginx/typemill_access.log;
    error_log   /var/log/nginx/typemill_error.log;

    # Defining what any directory's index file is going to look like
    index       index.php;

    # Set up robots.txt
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # --- EVERYTHING BELOW THIS LINE CAN BE LEFT ALONE

    # This enables PHP in your website
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        /etc/nginx/fastcgi_params;
    }

    # This makes sure PHP query URLs don't break. Usually.
    location / {
        try_files $uri $uri/ /index.php?$args;
        rewrite (.*?)index\.php/*(.*) /$1$2 redirect;
        rewrite (^|/)\.(?!well-known\/) /index.php break;
        rewrite ^/(system|content|data|settings|(media\/files\/)) /index.php break;
    }

    # This makes sure that missing links to image files
    # don't clog up your logs.
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires max;
        log_not_found off;
    }

    # The following two rule sets deny direct access to certain files
    # and kinds of files to prevent security issues.

    location ~\.(git|txt|md|yml|md|php|twig)$ {
        deny all;
        return 404;
    }

    location ~ ^/(licence\.md|readme\.md|composer\.lock|composer\.json|\.htaccess)$ {
        deny all;
        return 404;
    }
}
```

