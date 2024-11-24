# Forgejo (Git) server, running on Debian

## Table of contents

- [Install dependencies](#install-dependencies)
- [Configure nginx](#configure-nginx)
- [Configure Forgejo](#configure-forgejo)
- [Setting up Forgejo GUI](#setting-up-forgejo-gui)
- [References](#references)

## Install dependencies

Install dependencies:

```
nala install git git-lfs
```

## Configure nginx

Install nginx:

```
nala install nginx
systemctl enable nginx
systemctl start nginx
```

Do the certs. Either it's local, and the certs are self-signed; or, the server is public, use `certbot`.

```
nala install python3-certbot-nginx
sudo certbot --nginx
```

Now, if public, throw up firewall rules so that only the user can access the server. Definitely don't want a random to set up the server on the user's behalf.

Configure the `/etc/nginx/conf.d/forgejo.conf` file:

```
server {
    listen 80; # Listen on IPv4 port 80
    listen [::]:80; # Listen on IPv6 port 80
    return 301 https://<$SERVER>$request_uri;

    server_name <$SERVER>; # Change this to the server domain name.
}

server {
    listen 443 ssl;
    listen [::]:443 ;

    server_name <$SERVER>; # Change this to the server domain name.

    ssl_certificate /etc/nginx/ssl/forgejo.crt;
    ssl_certificate_key /etc/nginx/ssl/forgejo.key;

    location / {
        proxy_pass http://127.0.0.1:3000; # Port 3000 is the default Forgejo port

        proxy_set_header Connection $http_connection;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        client_max_body_size 512M;
    }
}
```

If using `certbot` certs, swap the `ssl_certificate` and `ssl_certificate_key` with:

```
ssl_certificate /etc/letsencrypt/live/<$DOMAIN>/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/<$DOMAIN>/privkey.pem; # managed by Certbot
include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
```

## Configure Forgejo

Create binary directory and change directory into it:

```
mkdir -pv ~/.local/src/forgejo-bin
cd ~/.local/src/forgejo-bin
```

Set up version:

```
# as of 2024 November 13, current version is 9.0.1
# as of 2024 November 24, current version is 9.0.2
VERSION="9.0.2"
```

Download binary:

```
curl -LJO https://codeberg.org/forgejo/forgejo/releases/download/v$VERSION/forgejo-$VERSION-linux-amd64
curl -LJO https://codeberg.org/forgejo/forgejo/releases/download/v$VERSION/forgejo-$VERSION-linux-amd64.asc
curl -LJO https://openpgpkey.forgejo.org/.well-known/openpgpkey/forgejo.org/hu/dj3498u4hyyarh35rkjfnghbjxug6b19
```

GPG verify:

```
gpg --import dj3498u4hyyarh35rkjfnghbjxug6b19.asc
gpg --verify forgejo-$VERSION-linux-amd64.asc forgejo-$VERSION-linux-amd64
```

Migrate binary and set permissions:

```
sudo cp forgejo-$VERSION-linux-amd64 /usr/local/bin/forgejo
sudo chmod 755 /usr/local/bin/forgejo
```

```
sudo useradd -m git
sudo mkdir -pv /var/lib/forgejo /etc/forgejo
sudo chown git: /var/lib/forgejo && sudo chmod 750 /var/lib/forgejo
sudo chown root:git /etc/forgejo && sudo chmod 770 /etc/forgejo
```

Edit the systemd file:

```
sudo vim /etc/systemd/system/forgejo.service
```

Systemd file - [source](https://codeberg.org/forgejo/forgejo/src/branch/forgejo/contrib/systemd/forgejo.service):

```
[Unit]
Description=Forgejo (Beyond coding. We forge.)
After=syslog.target
After=network.target
###
# Don't forget to add the database service dependencies
###
#
#Wants=mysql.service
#After=mysql.service
#
#Wants=mariadb.service
#After=mariadb.service
#
#Wants=postgresql.service
#After=postgresql.service
#
#Wants=memcached.service
#After=memcached.service
#
#Wants=redis.service
#After=redis.service
#
###
# If using socket activation for main http/s
###
#
#After=forgejo.main.socket
#Requires=forgejo.main.socket
#
###
# (You can also provide forgejo an http fallback and/or ssh socket too)
#
# An example of /etc/systemd/system/forgejo.main.socket
###
##
## [Unit]
## Description=Forgejo Web Socket
## PartOf=forgejo.service
##
## [Socket]
## Service=forgejo.service
## ListenStream=<some_port>
## NoDelay=true
##
## [Install]
## WantedBy=sockets.target
##
###

[Service]
# Uncomment the next line if you have repos with lots of files and get a HTTP 500 error because of that
# LimitNOFILE=524288:524288
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/forgejo/
# If using Unix socket: tells systemd to create the /run/forgejo folder, which will contain the forgejo.sock file
# (manually creating /run/forgejo doesn't work, because it would not persist across reboots)
#RuntimeDirectory=forgejo
ExecStart=/usr/local/bin/forgejo web --config /etc/forgejo/app.ini
Restart=always
Environment=USER=git HOME=/home/git FORGEJO_WORK_DIR=/var/lib/forgejo
# If you install Git to directory prefix other than default PATH (which happens
# for example if you install other versions of Git side-to-side with
# distribution version), uncomment below line and add that prefix to PATH
# Don't forget to place git-lfs binary on the PATH below if you want to enable
# Git LFS support
#Environment=PATH=/path/to/git/bin:/bin:/sbin:/usr/bin:/usr/sbin
# If you want to bind Forgejo to a port below 1024, uncomment
# the two values below, or use socket activation to pass Forgejo its ports as above
###
#CapabilityBoundingSet=CAP_NET_BIND_SERVICE
#AmbientCapabilities=CAP_NET_BIND_SERVICE
###
# In some cases, when using CapabilityBoundingSet and AmbientCapabilities option, you may want to
# set the following value to false to allow capabilities to be applied on Forgejo process. The following
# value if set to true sandboxes Forgejo service and prevent any processes from running with privileges
# in the host user namespace.
###
#PrivateUsers=false
###

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```
systemctl enable forgejo
systemctl start forgejo
systemctl status forgejo
```

## Setting up Forgejo GUI

Now, go to the website and do the install. Only things to change right now are:

- Database = sqlite
- Title
- Slogan
- Disable OpenID
- Create admin account

After doing the install, shut it down:

```
systemctl stop forgejo
```

Modify the permissions:

```
sudo chmod 750 /etc/forgejo && sudo chmod 640 /etc/forgejo/app.ini
```

Edit the `/etc/forgejo/app.ini` file and add the following:

```
[repository]
ROOT = /var/lib/forgejo/data/forgejo-repositories
DEFAULT_BRANCH = master

[repository.upload]
;; FILE_MAX_SIZE - units = MB, default 3 (MB)
FILE_MAX_SIZE = 4095
;; MAX_FILES - default 5
MAX_FILES = 1000

[server]
;; LFS_HTTP_AUTH_EXPIRY - default 20 minutes
LFS_HTTP_AUTH_EXPIRY = 180m

[git]
HOME_PATH = /home/git

;; all of these are 10x the defaults
[git.timeout]
DEFAULT = 3600
MIGRATE = 6000
MIRROR  = 3000
CLONE   = 3000
PULL    = 3000
GC      = 600
```

## References

- [Forgejo - Installation from binary](https://forgejo.org/docs/latest/admin/installation-binary/)
    - Forgejo docs on installing Forgejo from binary
