# YOURLS (Your Own URL Shortener)

[YOURLS](https://github.com/YOURLS/YOURLS) is a self-hostable URL shortener, written in PHP.

Update package repositories and install packages:

```
nala update && nala upgrade
nala install -y git mariadb-server nginx php php-bcmath php-cli php-curl php-fpm php-gd php-json php-mbstring php-mysql php-pear php-xml php-zip unzip wget
```

Set up the requisite services:

```
systemctl stop apache2
systemctl disable apache2
systemctl enable nginx
systemctl start nginx
systemctl enable mariadb
systemctl start mariadb
```

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

Download the YOURLS `git` repo:

```
cd /var/www
sudo git clone https://github.com/YOURLS/YOURLS.git
cd YOURLS/user/
sudo cp config-sample.php config.php
```

Configure YOURLS:

```
sudo vim config.php
```

Change the following lines. Note that the "YOURLS_SITE" entry can be an IP address (http://IPADDRESS) if DNS is not configured:

```
/** MySQL database username */
define( 'YOURLS_DB_USER', 'yourls' );

/** MySQL database password */
define( 'YOURLS_DB_PASS', 'password' );

/** The name of the database for YOURLS
 ** Use lower case letters [a-z], digits [0-9] and underscores [_] only */
define( 'YOURLS_DB_NAME', 'yourls_db' );

/** MySQL hostname.
 ** If using a non standard port, specify it like 'hostname:port', eg. 'localhost:9999' or '127.0.0.1:666' */
define( 'YOURLS_DB_HOST', 'localhost' );

/** MySQL tables prefix
 ** YOURLS will create tables using this prefix (eg `yourls_url`, `yourls_options`, ...)
 ** Use lower case letters [a-z], digits [0-9] and underscores [_] only */
define( 'YOURLS_DB_PREFIX', 'yourls_' );

define( 'YOURLS_SITE', 'http://yourls.local' );
$yourls_user_passwords = array(
        'admin' => 'adminpassword',
```

In addition, whenever YOURLS "is run", it will hash the passwords in the config file. If a new account is created, upon any user login or page refresh, YOURLS will hash the passwords.

Change permissions:

```
sudo chown -R www-data: /var/www/YOURLS
sudo chmod -R 775 /var/www/YOURLS
```

Edit `nginx`:

```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/yourls.conf
sudo vim /etc/nginx/sites-available/yourls.conf
```

Edit the file to contain this block. "server_name" can be either a domain, or IP, or both:

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
    fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    include fastcgi_params;
  }
}
```

Enable the site's `nginx` config:

```
sudo ln -s /etc/nginx/sites-available/yourls.conf /etc/nginx/sites-enabled/
systemctl restart nginx
```

Navigate to http://yourls.local/admin or http://IPADDRESS/admin and install YOURLS via the web interface. Only one of the two will work, and that's the one that was entered into "/var/www/YOURLS/user/config.php"

## References

- [YOURLS official documentation](https://yourls.org/docs)
