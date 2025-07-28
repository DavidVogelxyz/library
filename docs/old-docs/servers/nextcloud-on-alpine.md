# Nextcloud cloud storage, running on Alpine Linux

NB: This guide has been tested using VMs hosted on Vultr and Linode.

## Table of contents

- [Initial configuration](#initial-configuration)
- [Configuring UFW](#configuring-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Installing Nextcloud](#installing-nextcloud)
    - [PostgreSQL - database](#postgresql---database)
    - [Configuring SSL and HTTPS connections](#configuring-ssl-and-https-connections)
    - [Configuring nginx](#configuring-nginx)
    - [Configuring php](#configuring-php)
    - [Starting Nextcloud](#starting-nextcloud)
    - [Configuring the web admin user](#configuring-the-web-admin-user)
- [Enabling video communication](#enabling-video-communication)
- [References](#references)

## Initial configuration

Follow this guide on [configuring a server running Alpine Linux](/servers/configuring-alpine-server.md) to set up a new user account and secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

## Configuring UFW

Depending on the network's setup, it may also make sense to include a software firewall, such as `ufw`. For a publicly accessible server, it is highly advisable to configure `ufw` such that the available ports are limited as much as possible.

Add a default firewall rule to deny incoming connections:

```
sudo ufw default deny incoming
```

Add a default firewall rule to allow outgoing connections:

```
sudo ufw default allow outgoing
```

Set logging to off:

```
sudo ufw logging off
```

For generalized firewall rules, see [general firewall rules](#general-firewall-rules); to create firewall rules that only allow a certain IP to access the services, see [IP-specific firewall rules](#ip-specific-firewall-rules).

### General firewall rules

Add a firewall rule to allow SSH connections:

```
sudo ufw allow ssh
```

Add a firewall rule to allow HTTP connections:

```
sudo ufw allow http
```

Add a firewall rule to allow HTTPS connections:

```
sudo ufw allow https
```

### IP-specific firewall rules

Add a firewall rule to allow SSH connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 22
```

Add a firewall rule to allow HTTP connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 80
```

Add a firewall rule to allow HTTPS connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 443
```

### Enabling UFW

Enable `ufw`:

```
sudo ufw enable
```

Enable `ufw` to run on startup:

```
rc-update add ufw
```

To check and confirm the firewall rules, use the following command:

```
sudo ufw status
```

## Installing Nextcloud

Now, update the package repositories, and install all other relevant packages for Nextcloud, including:

- PostgreSQL, as the database
- Nextcloud's webserver component
- `nginx` and php-related components
- Nextcloud's default apps
- `ffmpeg`, for video thumbnail rendering
- `openssl`, for SSL certificates

```
apk update && apk add nextcloud-pgsql postgresql postgresql-client nextcloud-initscript nginx php81-fpm php81-opcache nextcloud-default-apps ffmpeg openssl
```

### PostgreSQL - database

Perform the initial setup for the PostgreSQL database with:

```
service postgresql setup
```

Start the PostgreSQL service with:

```
service postgresql start
```

Enter PostgreSQL with the following command:

```
sudo psql -U postgres
```

Now, create a user within PostgreSQL and set the password. Be sure to change `mycloud` and `test123` with more secure entries.

```
CREATE USER mycloud WITH PASSWORD 'test123';
```

Allow the new PostgreSQL user to create a database with:

```
ALTER ROLE mycloud CREATEDB;
```

Use the following command to exit PostgreSQL:

```
\q
```

Enable PostgreSQL to run on startup with:

```
rc-update add postgresql
```

### Configuring SSL and HTTPS connections

Now, it's time to set up some self-signed SSL certificates in order to enable HTTPS connections. Create the following directory and change directory into it.

```
sudo mkdir -pv /etc/nginx/ssl && cd /etc/nginx/ssl
```

Using the following two commands, create the required files.

Replace `$SERVER` with a name for the files being generated. This will likely be a name like "nextcloud" or "server".

The first command creates the server's private key, as well as a "certificate signing request" (CSR) file.

```
sudo openssl req -newkey rsa:4096 -nodes -keyout $SERVER.key -out $SERVER.csr
```

The next command uses the private key and the CSR to generate a certificate file.

```
sudo openssl x509 -signkey $SERVER.key -in $SERVER.csr -req -days 36500 -out $SERVER.crt
```

### Configuring nginx

Move the default `nginx` website conf file to a backup:

```
sudo mv /etc/nginx/http.d/default.conf /etc/nginx/http.d/default.conf.bak
```

Use a text editor (`vim`) to create the "/etc/nginx/http.d/nextcloud.conf" file:

```
sudo vim /etc/nginx/http.d/nextcloud.conf
```

Configure the file to look something similar to the below configuration file.

Obviously, swap out `$HOSTNAME` with the hostname of the server running nextcloud, and change `$SERVER` to the name of the '.crt' and '.key' files that were created in the previous step.

```
server {
    #listen       [::]:80; #uncomment for IPv6 support
    listen       80;
    return 301 https://$HOSTNAME$request_uri;
    server_name $HOSTNAME;
}

server {
    #listen       [::]:443 ssl; #uncomment for IPv6 support
    listen       443 ssl;
    server_name  $HOSTNAME;

    root /usr/share/webapps/nextcloud;
    index  index.php index.html index.htm;
    disable_symlinks off;

    ssl_certificate /etc/nginx/ssl/$SERVER.crt;
    ssl_certificate_key /etc/nginx/ssl/$SERVER.key;
    ssl_session_timeout  5m;

    #Enable Perfect Forward Secrecy and ciphers without known vulnerabilities
    #Beware! It breaks compatibility with older OS and browsers (e.g. Windows XP, Android 2.x, etc.)
    #ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA;
    #ssl_prefer_server_ciphers  on;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }
        #fastcgi_pass 127.0.0.1:9000;
        #fastcgi_pass unix:/run/php-fpm/socket;
        fastcgi_pass unix:/run/nextcloud/fastcgi.sock; # From the nextcloud-initscript package
            fastcgi_index index.php;
        include fastcgi.conf;
    }

    # Help pass nextcloud's configuration checks after install:
    # Per https://docs.nextcloud.com/server/22/admin_manual/issues/general_troubleshooting.html#service-discovery
    location ^~ /.well-known/carddav { return 301 /remote.php/dav/; }
    location ^~ /.well-known/caldav { return 301 /remote.php/dav/; }
    location ^~ /.well-known/webfinger { return 301 /index.php/.well-known/webfinger; }
    location ^~ /.well-known/nodeinfo { return 301 /index.php/.well-known/nodeinfo; }

# Spreed WebRTC
    location ^~ /webrtc {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_buffering             on;
        proxy_ignore_client_abort   off;
        proxy_redirect              off;
        proxy_connect_timeout       90;
        proxy_send_timeout          90;
        proxy_read_timeout          90;
        proxy_buffer_size           4k;
        proxy_buffers               4 32k;
        proxy_busy_buffers_size     64k;
        proxy_temp_file_write_size  64k;
        proxy_next_upstream         error timeout invalid_header http_502 http_503 http_504;
    }
}
```

Move the default `nginx` conf file to a backup:

```
sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

Use a text editor (`vim`) to create the "/etc/nginx/nginx.conf" file. Add the following:

```
# /etc/nginx/nginx.conf

user nginx;

# Set number of worker processes automatically based on number of CPU cores.
worker_processes auto;

# Enables the use of JIT for regular expressions to speed-up their processing.
pcre_jit on;

# Configures default error logger.
error_log /var/log/nginx/error.log warn;

# Includes files with directives to load dynamic modules.
include /etc/nginx/modules/*.conf;

# Include files with config snippets into the root context.
include /etc/nginx/conf.d/*.conf;

events {
    # The maximum number of simultaneous connections that can be opened by
    # a worker process.
    worker_connections 1024;
}

http {
    # Includes mapping of file name extensions to MIME types of responses
    # and defines the default type.
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Name servers used to resolve names of upstream servers into addresses.
    # It's also needed when using tcpsocket and udpsocket in Lua modules.
    #resolver 1.1.1.1 1.0.0.1 2606:4700:4700::1111 2606:4700:4700::1001;

    # Don't tell nginx version to the clients. Default is 'on'.
    server_tokens off;

    # Specifies the maximum accepted body size of a client request, as
    # indicated by the request header Content-Length. If the stated content
    # length is greater than this size, then the client receives the HTTP
    # error code 413. Set to 0 to disable. Default is '1m'.
    client_max_body_size 0;

    # Sendfile copies data between one FD and other from within the kernel,
    # which is more efficient than read() + write(). Default is off.
    sendfile on;

    # Causes nginx to attempt to send its HTTP response head in one packet,
    # instead of using partial frames. Default is 'off'.
    tcp_nopush on;

    # Enables the specified protocols. Default is TLSv1 TLSv1.1 TLSv1.2.
    # TIP: If you're not obligated to support ancient clients, remove TLSv1.1.
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;

    # Path of the file with Diffie-Hellman parameters for EDH ciphers.
    # TIP: Generate with: `openssl dhparam -out /etc/ssl/nginx/dh2048.pem 2048`
    #ssl_dhparam /etc/ssl/nginx/dh2048.pem;

    # Specifies that our cipher suits should be preferred over client ciphers.
    # Default is 'off'.
    ssl_prefer_server_ciphers on;

    # Enables a shared SSL cache with size that can hold around 8000 sessions.
    # Default is 'none'.
    ssl_session_cache shared:SSL:2m;

    # Specifies a time during which a client may reuse the session parameters.
    # Default is '5m'.
    ssl_session_timeout 1h;

    # Disable TLS session tickets (they are insecure). Default is 'on'.
    ssl_session_tickets off;

    # Enable gzipping of responses.
    #gzip on;

    # Set the Vary HTTP header as defined in the RFC 2616. Default is 'off'.
    gzip_vary on;

    # Helper variable for proxying websockets.
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    # Specifies the main log format.
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" "$http_x_forwarded_for"';

    # Sets the path, format, and configuration for a buffered log write.
    access_log /var/log/nginx/access.log main;

    # Includes virtual hosts configs.
    include /etc/nginx/http.d/*.conf;

    # for video chat
    map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
    }
}
```

### Configuring php

Use a text editor (`vim`) to edit the "/etc/php81/php.ini" file. Change `upload_max_filesize = 2M` to `upload_max_filesize = 513M`.

The Alpine Linux wiki article for Nextcloud suggests doing the following to improve performance.

Uncomment and edit the following lines in "/etc/php81/php.ini":

```
...
opcache.enable=1
opcache.enable_cli=1 // change from 0
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128 //you can reduce this slightly when short on RAM
opcache.save_comments=1
opcache.revalidate_freq=1 // change from 2
...
```

***DON'T DO THIS EDIT, IGNORE FOR NOW*** - Use a text editor (`vim`) to create the "/etc/php81/php-fpm.d/nextcloud.conf" file. Add:

```
; Maximal size of a file that can be uploaded via web interface.
php_admin_value[memory_limit] = 512M
php_admin_value[post_max_size] = 513M
php_admin_value[upload_max_filesize] = 513M
```

***DON'T DO THIS EDIT, IGNORE FOR NOW*** - Skipped some stuff about large files. Check with Alpine Linux wiki on Nextcloud for more info.

### Starting Nextcloud

Start services with:

```
service nginx start && service nextcloud start
```

Enable `nginx` and `nextcloud` to run on startup:

```
rc-update add nginx && rc-update add nextcloud
```

### Configuring the web admin user

Log into the web page at the domain and create an web admin user.

For PostgreSQL, the default port is `5432`.

Enter PostgreSQL with the following command:

```
sudo psql -U postgres
```

Disallow the PostgreSQL user from creating a database with:

```
ALTER ROLE mycloud NOCREATEDB;
```

Use the following command to exit PostgreSQL:

```
\q
```

## Enabling video communication

NB: This section still needs to be completed and tested in a live environment.

Add the following to the "/etc/nginx/nginx.conf" file, in the **HTTP** section:

```
# for video chat
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}
```

Reload `nginx`:

```
service nginx reload
```

Install Spreed WebRTC server -- follow up with the Alpine Linux wiki for Nextcloud

## References

- [Alpine Linux wiki - Nextcloud](https://wiki.alpinelinux.org/wiki/Nextcloud)
