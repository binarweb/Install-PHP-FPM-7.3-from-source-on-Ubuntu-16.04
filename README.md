# Installing PHP-FPM 7.3 from source on Ubuntu 16.04

This tutorial will get you started with nGinx server, PHP-FPM server and MySQL server.  

## Install dependencies

```bash
apt update
apt install git build-essential autoconf re2c bison libxml2-dev -y
apt install libssl-dev pkg-config zlib1g-dev libsodium-dev libcurl4-openssl-dev libjpeg-turbo8-dev libbz2-dev libpng++-dev libfreetype6-dev libzip-dev -y
apt install mysql-server nginx -y
```

## Download PHP 7

```bash
wget https://www.php.net/distributions/php-7.3.11.tar.gz -O /usr/src/php-7.3.11.tar.gz
cd /usr/src ; tar xfvz php-7.3.11.tar.gz ; cd php-7.3.11
```

## Compile PHP 7

```bash
./configure \
	--prefix=/usr/local/php7 \
	--with-config-file-path=/usr/local/php7/etc \
	--with-config-file-scan-dir=/usr/local/php7/etc/conf.d \
	--enable-bcmath \
	--with-bz2 \
	--with-curl \
	--enable-filter \
	--enable-fpm \
	--with-gd \
	--with-freetype-dir \
	--with-jpeg-dir \
	--with-png-dir \
	--enable-intl \
	--enable-mbstring \
	--enable-mysqlnd --with-mysql-sock=/var/lib/mysql/mysql.sock --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
	--with-pdo-sqlite \
	--disable-phpdbg --disable-phpdbg-webhelper \
	--enable-opcache \
	--with-openssl \
	--enable-simplexml \
	--with-sqlite3 \
	--enable-xmlreader --enable-xmlwriter \
	--enable-zip \
	--with-zlib \
	--with-fpm-user=www-data --with-fpm-group=www-data
make -j 4
```

where 4 is the number of processor cores  

## Install PHP 7

```bash
make install
```

The output will be something like this:  

```text
Installing shared extensions:     /usr/local/php7/lib/php/extensions/no-debug-non-zts-20180731/
Installing PHP CLI binary:        /usr/local/php7/bin/
Installing PHP CLI man page:      /usr/local/php7/php/man/man1/
Installing PHP FPM binary:        /usr/local/php7/sbin/
Installing PHP FPM defconfig:     /usr/local/php7/etc/
Installing PHP FPM man page:      /usr/local/php7/php/man/man8/
Installing PHP FPM status page:   /usr/local/php7/php/php/fpm/
Installing PHP CGI binary:        /usr/local/php7/bin/
Installing PHP CGI man page:      /usr/local/php7/php/man/man1/
Installing build environment:     /usr/local/php7/lib/php/build/
Installing header files:          /usr/local/php7/include/php/
Installing helper programs:       /usr/local/php7/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php7/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment:      /usr/local/php7/lib/php/
[PEAR] Archive_Tar    - installed: 1.4.7
[PEAR] Console_Getopt - installed: 1.4.2
[PEAR] Structures_Graph- installed: 1.1.1
[PEAR] XML_Util       - installed: 1.4.3
[PEAR] PEAR           - installed: 1.10.9
Wrote PEAR system config file at: /usr/local/php7/etc/pear.conf
You may want to add: /usr/local/php7/lib/php to your php.ini include_path
/usr/src/php-7.3.11/build/shtool install -c ext/phar/phar.phar /usr/local/php7/bin
ln -s -f phar.phar /usr/local/php7/bin/phar
Installing PDO headers:           /usr/local/php7/include/php/ext/pdo/
```

## Test the installed PHP version

```bash
php-fpm --version
```

## Copy php.ini from PHP source

```bash
cp /usr/src/php-7.3.11/php.ini-development /usr/local/php7/etc/php.ini
```

## Create a demo index.php file

```bash
nano /var/www/html/index.php
```
paste in the following:  

```php
<?php

phpinfo();
```

## Edit the nGinx configuration

```bash
nano /etc/nginx/sites-available/default
```

paste in the following:

```text
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.php index.html index.htm;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php7.0-cgi alone:
                fastcgi_pass 127.0.0.1:9000;
        #       # With php7.0-fpm:
        #       fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        location ~ /\.ht {
                deny all;
        }
}
```

then

```bash
service nginx restart
```

## Install PHP-FPM as a system service

```bash
mkdir /run/php-fpm/
```

```bash
nano /usr/lib/systemd/user/phpfpm.service
```

paste in the following:

```text
[Unit]
Description=The PHP FastCGI Process Manager
After=syslog.target network.target

[Service]
Type=simple
PIDFile=/run/php-fpm/php-fpm.pid
ExecStart=/usr/sbin/php-fpm --nodaemonize --fpm-config /usr/local/php7/etc/php-fpm.d/www.conf.default
ExecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target
```

## Install and start the PHP-FPM service

```bash
systemctl daemon-reload
systemctl enable phpfpm.service
systemctl start phpfpm.service
```

## Check the status of php-fpm service

```bash
systemctl status phpfpm.service
```

### Rerences:
https://linuxadmin.io/php-fpm-php7-source/  
https://github.com/gitKearney/php7-from-scratch  
https://serverfault.com/questions/785502/create-daemon-on-ubuntu-16-04  
https://www.digitalocean.com/community/questions/convert-run-at-startup-script-from-upstart-to-systemd-for-ubuntu-16
