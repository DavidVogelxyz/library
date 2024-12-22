# How to set up Snipe-IT on Debian

NB: this guide references the IP address `127.0.0.1`. Obviously, this is a reference to the hostname `localhost`. Either can be used in place of the other; however, using the IP address itself avoids both the "host file lookup", as well as the potential for trying the IPv6 entry for `localhost`.

## Introduction

[Snipe-IT](https://github.com/snipe/snipe-it) is a self-hostable inventory management tool, written in PHP.

This guide assumes that Snipe-IT is being installed on a Debian server, with `nala` already installed. Also, it is assumed that the user installing Snipe-IT has certain aliases set up, such that commands like `nala` and `systemctl` can be run without `sudo` in front of the commands.

## Table of contents

- [Introduction](#introduction)
- [Initial configuration](#initial-configuration)
- [Preparing the server for Snipe-IT](#preparing-the-server-for-snipe-it)
- [Configuring database](#configuring-database)
- [Installing Snipe-IT](#installing-snipe-it)
- [References](#references)

## Initial configuration

For notes on pre-configuration, refer to the guide on [configuring a server running Debian](configuring-debian-server.md).

Set up `zsh` as described in the [Bitcoin node guide](bitcoin-node-on-debian/initial-configuration.md#configuring-zsh):

```
nala install -y bat fzf lf stow ueberzug zsh zsh-syntax-highlighting
mkdir -pv ~/.cache/zsh ~/.local/bin
chsh -s /bin/zsh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.local/src/powerlevel10k
```

The `fonts.sh` script for the `zsh` prompt:

```
fonts=(
"MesloLGS_NF_Regular.ttf"
"MesloLGS_NF_Bold.ttf"
"MesloLGS_NF_Italic.ttf"
"MesloLGS_NF_Bold_Italic.ttf"
)

echo "There are $(echo ${#fonts[@]}) fonts to install. It shouldn't take long."
echo "Elevated privileges are required to properly install the fonts. Please enter your password if asked."

sudo mkdir -p /usr/local/share/fonts/m

for font in "${fonts[@]}"; do
    webfont=$(echo $font | sed 's/_/%20/g')
    [ ! -e /usr/local/share/fonts/m/$font ] \
        && echo "now installing $font ..." \
        && sudo curl -L https://github.com/romkatv/powerlevel10k-media/raw/master/$webfont -o /usr/local/share/fonts/m/$font > /dev/null 2>&1
done
```

Additional commands to set up `zsh` and `lf`:

```
cd ~/.dotfiles && stow .
sed -i "s/'~\/.config\/lf\/scope/'~\/.config\/lf\/scope-debian/g" ~/.config/lf/lfrc
```

## Preparing the server for Snipe-IT

Download `php` dependencies (taken from YOURLS guide):

```
nala install -y git mariadb-server nginx php php-bcmath php-cli php-curl php-fpm php-gd php-json php-ldap php-mbstring php-mysql php-pear php-xml php-zip unzip wget
```

Stop `apache2`:

```
systemctl stop apache2 && systemctl disable apache2
```

Start `nginx`:

```
systemctl enable nginx && systemctl start nginx
```

## Configuring database

Start `mariadb`:

```
systemctl enable mariadb && systemctl start mariadb
```

Secure `mariadb`:

```
sudo mysql_secure_installation
```

Set up `mariadb`:

```
sudo mysql
```

MySQL commands:

```
CREATE DATABASE snipe_db;
GRANT ALL PRIVILEGES ON snipe_db.* TO 'snipe'@'127.0.0.1' IDENTIFIED BY "password";
FLUSH PRIVILEGES;
exit;
```

## Installing Snipe-IT

Clone the `snipe-it` repo:

```
git clone --depth=1 https://github.com/snipe/snipe-it ~/.local/src/snipe-it
```

Set up the `.env` file:

```
cd ~/.local/src/snipe-it
cp .env.example .env
vim .env
```

The `.env` file should look like:

```
# --------------------------------------------
# REQUIRED: BASIC APP SETTINGS
# --------------------------------------------
APP_ENV=production
APP_DEBUG=false
APP_KEY=<APP_KEY>
APP_URL=<APP_URL>
APP_FORCE_TLS=false
APP_TIMEZONE='UTC'
APP_LOCALE='en-US'
MAX_RESULTS=500

# --------------------------------------------
# REQUIRED: UPLOADED FILE STORAGE SETTINGS
# --------------------------------------------
PRIVATE_FILESYSTEM_DISK=local
PUBLIC_FILESYSTEM_DISK=local_public

#PRIVATE_FILESYSTEM_DISK=s3_private
#PUBLIC_FILESYSTEM_DISK=s3_public

# --------------------------------------------
# REQUIRED: DATABASE SETTINGS
# --------------------------------------------
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=<DB_DATABASE>
DB_USERNAME=<DB_USERNAME>
DB_PASSWORD=<DB_PASSWORD>
DB_PREFIX=null
DB_DUMP_PATH='/usr/bin'
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_unicode_ci
DB_SANITIZE_BY_DEFAULT=false

# --------------------------------------------
# OPTIONAL: SSL DATABASE SETTINGS
# --------------------------------------------
DB_SSL=false
DB_SSL_IS_PAAS=false
DB_SSL_KEY_PATH=null
DB_SSL_CERT_PATH=null
DB_SSL_CA_PATH=null
DB_SSL_CIPHER=null
DB_SSL_VERIFY_SERVER=null

# --------------------------------------------
# REQUIRED: OUTGOING MAIL SERVER SETTINGS
# --------------------------------------------
MAIL_MAILER=smtp
MAIL_HOST=email-smtp.us-west-2.amazonaws.com
MAIL_PORT=587
MAIL_USERNAME=YOURUSERNAME
MAIL_PASSWORD=YOURPASSWORD
MAIL_FROM_ADDR=you@example.com
MAIL_FROM_NAME='Snipe-IT'
MAIL_REPLYTO_ADDR=you@example.com
MAIL_REPLYTO_NAME='Snipe-IT'
MAIL_AUTO_EMBED_METHOD='attachment'
MAIL_TLS_VERIFY_PEER=true

# MAIL_ENCRYPTION is no longer supported. SymfonyMailer will use tls if it's
# advertised, and won't if it's not. If you want to use your mail server's IP but it's failing
# because of certificate errors, set MAIL_TLS_VERIFY_PEER-true

# --------------------------------------------
# REQUIRED: IMAGE LIBRARY
# This should be gd or imagick
# --------------------------------------------
IMAGE_LIB=gd

# --------------------------------------------
# OPTIONAL: BACKUP SETTINGS
# --------------------------------------------
MAIL_BACKUP_NOTIFICATION_DRIVER=null
MAIL_BACKUP_NOTIFICATION_ADDRESS=null
BACKUP_ENV=true
ALLOW_BACKUP_DELETE=false
ALLOW_DATA_PURGE=false

# --------------------------------------------
# OPTIONAL: SESSION SETTINGS
# --------------------------------------------
SESSION_DRIVER=file
SESSION_LIFETIME=12000
EXPIRE_ON_CLOSE=false
ENCRYPT=false
COOKIE_NAME=snipeit_session
PASSPORT_COOKIE_NAME='snipeit_passport_token'
COOKIE_DOMAIN=null
SECURE_COOKIES=false
API_TOKEN_EXPIRATION_YEARS=15
BS_TABLE_STORAGE=cookieStorage
BS_TABLE_DEEPLINK=true

# --------------------------------------------
# OPTIONAL: SECURITY HEADER SETTINGS
# --------------------------------------------
APP_TRUSTED_PROXIES=192.168.1.1,10.0.0.1
ALLOW_IFRAMING=false
REFERRER_POLICY=same-origin
ENABLE_CSP=false
ADDITIONAL_CSP_URLS=null
CORS_ALLOWED_ORIGINS=null
ENABLE_HSTS=false

# --------------------------------------------
# OPTIONAL: CACHE SETTINGS
# --------------------------------------------
CACHE_DRIVER=file
QUEUE_DRIVER=sync
CACHE_PREFIX=snipeit

# --------------------------------------------
# OPTIONAL: REDIS SETTINGS
# --------------------------------------------
REDIS_HOST=null
REDIS_PASSWORD=null
REDIS_PORT=null

# --------------------------------------------
# OPTIONAL: MEMCACHED SETTINGS
# --------------------------------------------
MEMCACHED_HOST=null
MEMCACHED_PORT=null

# --------------------------------------------
# OPTIONAL: PUBLIC S3 Settings
# --------------------------------------------
PUBLIC_AWS_SECRET_ACCESS_KEY=null
PUBLIC_AWS_ACCESS_KEY_ID=null
PUBLIC_AWS_DEFAULT_REGION=null
PUBLIC_AWS_BUCKET=null
PUBLIC_AWS_URL=null
PUBLIC_AWS_BUCKET_ROOT=null

# --------------------------------------------
# OPTIONAL: PRIVATE S3 Settings
# --------------------------------------------
PRIVATE_AWS_ACCESS_KEY_ID=null
PRIVATE_AWS_SECRET_ACCESS_KEY=null
PRIVATE_AWS_DEFAULT_REGION=null
PRIVATE_AWS_BUCKET=null
PRIVATE_AWS_URL=null
PRIVATE_AWS_BUCKET_ROOT=null

# --------------------------------------------
# OPTIONAL: AWS Settings
# --------------------------------------------
AWS_ACCESS_KEY_ID=null
AWS_SECRET_ACCESS_KEY=null
AWS_DEFAULT_REGION=null

# --------------------------------------------
# OPTIONAL: LOGIN THROTTLING
# --------------------------------------------
LOGIN_MAX_ATTEMPTS=5
LOGIN_LOCKOUT_DURATION=60
LOGIN_AUTOCOMPLETE=false

# --------------------------------------------
# OPTIONAL: FORGOTTEN PASSWORD SETTINGS
# --------------------------------------------
RESET_PASSWORD_LINK_EXPIRES=15
PASSWORD_CONFIRM_TIMEOUT=10800
PASSWORD_RESET_MAX_ATTEMPTS_PER_MIN=50

# --------------------------------------------
# OPTIONAL: MISC
# --------------------------------------------
LOG_CHANNEL=single
LOG_MAX_DAYS=10
APP_LOCKED=false
APP_CIPHER=AES-256-CBC
APP_FORCE_TLS=false
APP_ALLOW_INSECURE_HOSTS=false
GOOGLE_MAPS_API=
LDAP_MEM_LIM=500M
LDAP_TIME_LIM=600
IMPORT_TIME_LIMIT=600
IMPORT_MEMORY_LIMIT=500M
REPORT_TIME_LIMIT=12000
REQUIRE_SAML=false
API_THROTTLE_PER_MINUTE=120
CSV_ESCAPE_FORMULAS=true
LIVEWIRE_URL_PREFIX=null

# --------------------------------------------
# OPTIONAL: HASHING
# --------------------------------------------
HASHING_DRIVER='bcrypt'
BCRYPT_ROUNDS=10
ARGON_MEMORY=1024
ARGON_THREADS=2
ARGON_TIME=2

# --------------------------------------------
# OPTIONAL: SCIM
# --------------------------------------------
SCIM_TRACE=false
SCIM_STANDARDS_COMPLIANCE=false
```

Configure the `.env` file by changing the following parameters:

- `APP_URL`
- `APP_FORCE_TLS`
- `APP_TIMEZONE`
- `DB_DATABASE`
- `DB_USERNAME`
- `DB_PASSWORD`

Install `composer`:

```
curl -sS https://getcomposer.org/installer | php
php composer.phar install --no-dev --prefer-source
```

Set up `php` app key:

```
php artisan key:generate
```

Back up `php` app key. Return to the `.env` file and save the new `APP_KEY` somewhere safe.

Check dependencies:

```
php composer.phar check-platform-reqs
```

Set up `nginx`:

```
server {
    listen 80;
    server_name <HOSTNAME>;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name <HOSTNAME>;

    root /home/snipe/.local/src/snipe-it/public/;
    index index.php index.html index.htm;

    ssl_certificate /etc/nginx/ssl/<SERVER>.crt;
    ssl_certificate_key /etc/nginx/ssl/<SERVER>.key;
    #ssl_session_timeout  5m;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri $uri/ =404;
        #fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
        include fastcgi_params;
    }
}
```

Replace `<HOSTNAME>` and `<SERVER>` with the appropriate values. Also, be sure that the `<SERVER>.crt` and `<SERVER>.key` can be found at the specified path.

Test `nginx`:

```
sudo ln -s /etc/nginx/sites-available/snipe-it /etc/nginx/sites-enabled/ && systemctl restart nginx
```

Change file permissions:

```
cd ~/.local/src
sudo chown -R www-data: snipe-it
```

Navigate to `<HOSTNAME>`, and the "pre-flight" webpage should appear!

Fixing a database migration issue:

```
php upgrade.php
php artisan migrate
```

## References

- [Snipe-IT - Installation](https://snipe-it.readme.io/docs/installation)
- [Snipe-IT - Using Nginx and PHP-FPM](https://snipe-it.readme.io/docs/linuxosx#using-nginx-and-php-fpm)
- [GitHub - snipe/snipe-it - 500 Error After Update](https://github.com/snipe/snipe-it/issues/15924#issuecomment-2536835474)