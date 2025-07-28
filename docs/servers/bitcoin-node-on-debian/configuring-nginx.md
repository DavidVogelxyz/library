# Configuring nginx

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Installing and configuring "nginx"](#installing-and-configuring-nginx)
- [References](#references)

## Introduction

`nginx` is a web server and reverse proxy, allowing the user to direct traffic flows to the node and host web pages, etc. This section of the guide will describe how to install and configure `nginx`.

## Installing and configuring "nginx"

Install `nginx` by running the following command:

```
nala install -y nginx
```

Configure an SSL certificate for the server:

```
sudo openssl req -x509 -nodes -newkey rsa:4096 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -subj "/CN=localhost" -days 365000
```

Open the `/etc/nginx/nginx.conf` file with a text editor:

```
sudo vim /etc/nginx/nginx.conf
```

Overwrite its contents with the following:

```
user www-data;
worker_processes 1;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

stream {
    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 4h;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    include /etc/nginx/streams-enabled/*.conf;
}
```

Create the following directory:

```
sudo mkdir /etc/nginx/streams-enabled
```

Test the `nginx` configuration file with the following command:

```
sudo nginx -t
```

If the test fails in reference to `unknown directive "stream"`, install the following package and test again:

```
nala install -y libnginx-mod-stream
```

## References

- [Raspibolt - Security](https://raspibolt.org/guide/raspberry-pi/security.html)
    - Reference for initial setup of `nginx` reverse proxy.
