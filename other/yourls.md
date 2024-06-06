# YOURLS (Your Own URL Shortener)

[YOURLS](https://github.com/YOURLS/YOURLS) is a self-hostable URL shortener, written in PHP.

This guide assumes that YOURLS is being installed on a Debian server, with `nala` already installed. Also, it is assumed that the user installing YOURLS has certain aliases set up, such that commands like `nala` and `systemctl` can be run without `sudo` in front of the commands.

## Table of contents

- [Initial setup](#initial-setup)
- [Configuring database](#configuring-database)
- [Installing YOURLS](#installing-yourls)
- [References](#references)

## Initial setup

Update package repositories:

```
nala update && nala upgrade
```

Install packages:

```
nala install -y git mariadb-server nginx php php-bcmath php-cli php-curl php-fpm php-gd php-json php-mbstring php-mysql php-pear php-xml php-zip unzip wget
```

Set up the requisite services.

First, stop and disable `apache2`.

```
systemctl stop apache2 && systemctl disable apache2
```

Next, enable and start `nginx`.

```
systemctl enable nginx && systemctl start nginx
```

Also, enable and start `mariadb`.

```
systemctl enable mariadb && systemctl start mariadb
```

## Configuring database

Secure the new `mysql` installation:

```
sudo mysql_secure_installation
```

Set up `mysql` with the following:

```
sudo mysql
```

Do this in `mysql`:

```
CREATE DATABASE yourls_db;
GRANT ALL PRIVILEGES ON yourls_db.* TO 'yourls'@'localhost' IDENTIFIED BY "password";
FLUSH PRIVILEGES;
exit;
```

## Installing YOURLS

Clone the YOURLS `git` repo:

```
cd /var/www && sudo git clone https://github.com/YOURLS/YOURLS.git
```

Copy over the sample `config.php` file:

```
cd YOURLS/user/ && sudo cp config-sample.php config.php
```

Configure YOURLS:

```
sudo vim config.php
```

Change the following lines. Note that the "YOURLS_SITE" entry can be an IP address (`http://$IPADDRESS`) if DNS is not configured:

```
define( 'YOURLS_DB_USER', 'yourls' );

define( 'YOURLS_DB_PASS', 'password' );

define( 'YOURLS_DB_NAME', 'yourls_db' );

define( 'YOURLS_DB_HOST', 'localhost' );

define( 'YOURLS_DB_PREFIX', 'yourls_' );

define( 'YOURLS_SITE', 'http://yourls.local' );

define( 'YOURLS_COOKIEKEY', 'A random secret hash used to encrypt cookies. You don't have to remember it, make it long and complicated' );

$yourls_user_passwords = array(
        'admin' => 'adminpassword',
```

In addition, whenever YOURLS "is run", it will hash the passwords in the config file. If a new account is created, upon any user login or page refresh, YOURLS will hash the passwords.

Change permissions:

```
sudo chown -R www-data: /var/www/YOURLS && sudo chmod -R 775 /var/www/YOURLS
```

Edit `nginx` configuration file:

```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/yourls.conf && sudo vim /etc/nginx/sites-available/yourls.conf
```

Edit the file to contain this block. `server_name` can be either a domain, or IP, or both:

```
server {
    listen 80;
    server_name yourls.local $IPADDRESS;
    root /var/www/YOURLS;
    index index.php index.html index.htm;
    location / {
        try_files $uri $uri/ /yourls-loader.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        include fastcgi_params;
    }
}
```

As a note: the `fastcgi_pass` parameter needs to reference the correct version of PHP found on the system. If the webpage doesn't appear after linking the site's `nginx` configuration in the next step, check the PHP version installed on the server against this value, and update it if necessary.

Enable the site's `nginx` configuration by adding a symlink to `/etc/nginx/sites-enabled`:

```
sudo ln -s /etc/nginx/sites-available/yourls.conf /etc/nginx/sites-enabled/ && systemctl restart nginx
```

Navigate to `http://yourls.local/admin` or `http://$IPADDRESS/admin` and install YOURLS via the web interface. Only one of the two will work, and that's the one that was entered into "/var/www/YOURLS/user/config.php"

## References

- [YOURLS official documentation](https://yourls.org/docs)
