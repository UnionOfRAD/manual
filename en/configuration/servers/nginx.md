# Configuring Nginx

Nginx requires a simple rewrite configuration that enables Lithium to serve dynamic URLs. With nginx, each domain has its own configuration file that encapsulates all such rewriting rules.

## Domain Specific Config

The following example file is typically stored at `/etc/nginx/sites-available/<domain.com>.conf`. All instances of `DOMAIN.COM` can be replaced with the domain name of your site or application.

{{{
server {
        listen   IP_ADDRESS_HERE:80;
        server_name DOMAIN.COM;

        root   /var/www/DOMAIN.COM/webroot/;
        access_log /var/log/DOMAIN.com/access.log;
        error_log /var/log/DOMAIN.com/error.log warn;

        index  index.php index.html;

        try_files $uri $uri/ /index.php?$args;

        location ~ \.php$
        {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index  index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include /etc/nginx/fastcgi_params;
        }
        location ~ /\.ht {
                deny all;
        }
}
}}}
