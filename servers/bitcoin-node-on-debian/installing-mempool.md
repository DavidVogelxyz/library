# Installing Mempool.space (Timechain explorer)

[Back to the home page](README.md)

## Table of contents

- [Introduction](#Introduction)
- [Installing Mempool](#Installing-Mempool)
- [Configuring Mempool](#Configuring-Mempool)
    - [Configuring the Mempool backend](#Configuring-the-Mempool-backend)
    - [Configuring the Mempool frontend](#Configuring-the-Mempool-frontend)
    - [Finalizing the Mempool configuration](#Finalizing-the-Mempool-configuration)
- [Adding a hidden service for Mempool](#Adding-a-hidden-service-for-Mempool)
- [References](#references)

## Introduction

[Mempool.space](https://mempool.space) is a Bitcoin timechain explorer and visualizer. While the link directs to a publicly hosted version of the site, it is possible to set up a self-hosted version on the Bitcoin node that references the node, preventing any forms of data leak.

## Installing Mempool

To install Mempool, perform the following steps.

First, install `nodejs`.

Note: the best way to go about this on Debian is **not** to use the `nodejs` package found in the Debian package repositories. Instead, use the following curl command to download and prepare a version of `nodejs` provided by the Node.js maintainers.

```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

Now, install `nodejs`:

```
nala install -y nodejs
```

To confirm that `nodejs` installed correctly, check the node's `node` version:

```
node -v
```

It is also a good idea to check the node's `npm` version at this time:

```
npm --version
```

Now, add a `ufw` firewall rule to allow connections to Mempool:

```
sudo ufw allow 4081/tcp comment 'allow Mempool SSL'
```

Confirm the rule was entered correctly with the following:

```
sudo ufw status
```

Next, create a new user to manage the Mempool service:

```
sudo useradd -G bitcoin -s /bin/bash -m mempool
```

Note that the `-G` option directs the `mempool` user to be added to the `bitcoin` group upon creation, and the `-s` option changes the user's default shell to `/bin/bash`. The `-m` option directs the command to create a home directory for the new user.

Switch users to the `mempool` user:

```
su - mempool
```

Clone the Mempool project into the `mempool` user's home directory, then change directory into it:

```
git clone https://github.com/mempool/mempool && cd mempool
```

Using the [Mempool's GitHub page](https://github.com/mempool/mempool) as a reference, direct the `git checkout` command to checkout the latest release version (as of writing, `v3.0.0`):

```
git checkout v3.0.0
```

Log out of the `mempool` user, and back to the main admin user.

## Configuring Mempool

As the main admin user, install the MariaDB database package:

```
nala update && nala install -y mariadb-server mariadb-client
```

Use the following command to generate a randomized 25 character password; this password will be used to secure the SQL database (later referred to as `<PASSWORD_GENERATED_MOMENTS_AGO>`):

```
tr -dc A-Za-z0-9 </dev/urandom | head -c 25; echo
```

Secure the new `mysql` installation by following the prompts:

```
sudo mysql_secure_installation
```

Now, enter the shell for the SQL database:

```
sudo mysql
```

Run the following commands:

```
CREATE DATABASE mempool;
GRANT ALL PRIVILEGES ON mempool.* TO 'mempool'@'127.0.0.1' IDENTIFIED BY "<PASSWORD_GENERATED_MOMENTS_AGO>";
FLUSH PRIVILEGES;
exit;
```

Switch users to the `mempool` user:

```
su - mempool
```

### Configuring the Mempool backend

Change directory into `~/mempool/backend`, then install and build the backend:

```
cd ~/mempool/backend && npm install --prod && npm run build
```

Create the configuration file in this directory by opening `mempool-config.json` in a text editor:

```
vim mempool-config.json
```

Add the following content to the file:

```
{
    "MEMPOOL": {
        "NETWORK": "mainnet",
            "BACKEND": "electrum",
            "HTTP_PORT": 8999,
            "SPAWN_CLUSTER_PROCS": 0,
            "API_URL_PREFIX": "/api/v1/",
            "POLL_RATE_MS": 2000,
            "CACHE_DIR": "./cache",
            "CLEAR_PROTECTION_MINUTES": 20,
            "RECOMMENDED_FEE_PERCENTILE": 50,
            "BLOCK_WEIGHT_UNITS": 4000000,
            "INITIAL_BLOCKS_AMOUNT": 8,
            "MEMPOOL_BLOCKS_AMOUNT": 8,
            "PRICE_FEED_UPDATE_INTERVAL": 3600,
            "USE_SECOND_NODE_FOR_MINFEE": false,
            "EXTERNAL_ASSETS": []
    },

    "CORE_RPC": {
        "HOST": "127.0.0.1",
        "PORT": 8332,
        "USERNAME": "<ADMIN_USERNAME>",
        "PASSWORD": "<BITCOIN_RPC_PASSWORD>"
    },

    "ELECTRUM": {
        "HOST": "127.0.0.1",
        "PORT": 50002,
        "TLS_ENABLED": true
    },

    "DATABASE": {
        "ENABLED": true,
        "HOST": "127.0.0.1",
        "PORT": 3306,
        "USERNAME": "mempool",
        "PASSWORD": "<MEMPOOL_SQL_DATABASE_PASSWORD>",
        "DATABASE": "mempool"
    },

    "SOCKS5PROXY": {
        "ENABLED": true,
        "USE_ONION": true,
        "HOST": "127.0.0.1",
        "PORT": 9050
    },

    "PRICE_DATA_SERVER": {
        "TOR_URL": "http://wizpriceje6q5tdrxkyiazsgu7irquiqjy2dptezqhrtu7l2qelqktid.onion/getAllMarketPrices"
    }
}
```

Note: remember to change `<ADMIN_USERNAME>` and `<BITCOIN_RPC_PASSWORD>` to the values set when configuring Bitcoin Core. Also, change `<MEMPOOL_SQL_DATABASE_PASSWORD>` to the password set earlier, when creating the SQL database.

With the file configured properly, start the backend and confirm that it's running as intended:

```
npm run start
```

After ~30 seconds, output like the following should appear:

```
Updating mempool
Mempool updated in 0.xyz seconds
```

Exit the `npm` process.

### Configuring the Mempool frontend

While still logged in as the `mempool` user, change directory into `~/mempool/frontend`, then install and build the frontend:

```
cd ~/mempool/frontend && npm install --prod && npm run build
```

Log out of the `mempool` user, and back to the main admin user.

Move the frontend output into the webroot directory, then change the owner to the `www-data` user:

```
sudo rsync -av --delete /home/mempool/mempool/frontend/dist/mempool/ /var/www/mempool/ && sudo chown -R www-data: /var/www/mempool
```

### Finalizing the Mempool configuration

Earlier, the file `mempool-config.json` was created, and it contains sensitive credentials to the Bitcoin Core RPC process. To limit who can see and interact with those credentials, change the permissions on the file so that only its owner (the `mempool` user) can read or write that file:

```
sudo chmod 600 /home/mempool/mempool/backend/mempool-config.json
```

Next, open the `/etc/nginx/sites-available/mempool-ssl.conf` file in a text editor:

```
sudo vim /etc/nginx/sites-available/mempool-ssl.conf
```

Add the following content to the file:

```
proxy_read_timeout 300;
proxy_connect_timeout 300;
proxy_send_timeout 300;

map $http_accept_language $header_lang {
    default en-US;
    ~*^en-US en-US;
    ~*^en en-US;
    ~*^ar ar;
    ~*^ca ca;
    ~*^cs cs;
    ~*^de de;
    ~*^es es;
    ~*^fa fa;
    ~*^fr fr;
    ~*^ko ko;
    ~*^it it;
    ~*^he he;
    ~*^ka ka;
    ~*^hu hu;
    ~*^mk mk;
    ~*^nl nl;
    ~*^ja ja;
    ~*^nb nb;
    ~*^pl pl;
    ~*^pt pt;
    ~*^ro ro;
    ~*^ru ru;
    ~*^sl sl;
    ~*^fi fi;
    ~*^sv sv;
    ~*^th th;
    ~*^tr tr;
    ~*^uk uk;
    ~*^vi vi;
    ~*^zh zh;
    ~*^hi hi;
}

map $cookie_lang $lang {
    default $header_lang;
    ~*^en-US en-US;
    ~*^en en-US;
    ~*^ar ar;
    ~*^ca ca;
    ~*^cs cs;
    ~*^de de;
    ~*^es es;
    ~*^fa fa;
    ~*^fr fr;
    ~*^ko ko;
    ~*^it it;
    ~*^he he;
    ~*^ka ka;
    ~*^hu hu;
    ~*^mk mk;
    ~*^nl nl;
    ~*^ja ja;
    ~*^nb nb;
    ~*^pl pl;
    ~*^pt pt;
    ~*^ro ro;
    ~*^ru ru;
    ~*^sl sl;
    ~*^fi fi;
    ~*^sv sv;
    ~*^th th;
    ~*^tr tr;
    ~*^uk uk;
    ~*^vi vi;
    ~*^zh zh;
    ~*^hi hi;
}

server {
    listen 4081 ssl;
    listen [::]:4081 ssl;
    server_name _;
    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_session_timeout 4h;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers on;

    include /etc/nginx/snippets/nginx-mempool.conf;
}
```

Next, create a symlink from the `sites-available` directory to the `sites-enabled` directory:

```
sudo ln -sf /etc/nginx/sites-available/mempool-ssl.conf /etc/nginx/sites-enabled/
```

Now, copy the Mempool nginx configuration file into the `/etc/nginx/snippets` directory:

```
sudo rsync -av /home/mempool/mempool/nginx-mempool.conf /etc/nginx/snippets
```

Back up the current `nginx` configuration file by moving it to a different path:

```
sudo mv -v /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

Open the `/etc/nginx/nginx.conf` in a text editor to create a new file:

```
sudo vim /etc/nginx/nginx.conf
```

Add the following contents to the file:

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers on;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    gzip on;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

stream {
    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 4h;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers on;
    include /etc/nginx/streams-enabled/*.conf;
}
```

Test the `nginx` configuration file with the following command:

```
sudo nginx -t
```

Now, use `systemctl reload` to update the `nginx` configuration:

```
systemctl reload nginx
```

Now, create a Mempool service file, so that the Mempool process can run automatically when the node reboots:


```
sudo vim /etc/systemd/system/mempool.service
```

Add the following content to the file:

```
# <HOSTNAME>: systemd unit for Mempool
# /etc/systemd/system/mempool.service

[Unit]
Description=mempool
After=bitcoind.service

[Service]
WorkingDirectory=/home/mempool/mempool/backend
ExecStart=/usr/bin/node --max-old-space-size=2048 dist/index.js
User=mempool

# Restart on failure but no more than default times (DefaultStartLimitBurst=5) every 10 minutes (600 seconds). Otherwise stop
Restart=on-failure
RestartSec=600

# Hardening measures
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true

[Install]
WantedBy=multi-user.target
```

Note: `<HOSTNAME>` should be the hostname of the node.

Enable the new Mempool service to start when the system boots, and start the service:

```
systemctl enable mempool && systemctl start mempool
```

Check the Mempool service logs to see it working:

```
sudo journalctl -fu mempool
```

Now, the site should be available at `https://<HOSTNAME.TLD>:4081`. In this case, `<HOSTNAME.TLD>` can also be the IP address of the node.

## Adding a hidden service for Mempool

As is described in the section on [adding a hidden service](#adding-a-hidden-service), follow these steps to create a hidden service for Mempool:

First, open the `/etc/tor/torrc` file in a text editor:

```
sudo vim /etc/tor/torrc
```

In the section marked as "just for location-hidden services", add the following lines:

```
# Hidden Service for Mempool
HiddenServiceDir /var/lib/tor/hidden_service_mempool
HiddenServiceVersion 3
HiddenServicePort 443 127.0.0.1:4081
```

Exit the file, and reload the `tor` configurations with the following command:

```
systemctl reload tor
```

Use `cat` to output the file contents in the corresponding directory to obtain the onion link to the service:

```
sudo cat /var/lib/tor/hidden_service_mempool/hostname
```

Now, it is possible to access Mempool remotely using Tor.

## References

- [Raspibolt - Mempool timechain explorer](https://raspibolt.org/guide/bonus/bitcoin/mempool.html)
    - Reference for initial setup of the Mempool timechain explorer.
