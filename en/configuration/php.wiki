# Configuring PHP
please, run make tests!

## PHP FPM
--enable-fpm
### when you configure php with fpm there are a few more steps to get everything setup. The commands below assume you installed PHP to `/usr/local/`
{{{
mkdir /var/log/php-fpm
chown -R www-data:www-data /var/log/php-fpm
cp -f php.ini-production /usr/local/etc/php.ini
chmod 644 /usr/local/etc/php.ini
cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
cp -f sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm

//example values for php-fpm.conf
pm.max_children = 150
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 10
}}}

## Apache
`--with-apxs2=/usr/sbin/apxs`


