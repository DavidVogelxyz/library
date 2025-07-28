# Configuring Tor

[Back to the home page](README.md)

## Table of contents

- [Introduction](#introduction)
- [Installing and configuring Tor](#installing-and-configuring-tor)
    - [Adding a hidden service](#adding-a-hidden-service)
- [References](#references)

## Introduction

Tor is an acronym for "The Onion Router", and is a network that allows users to anonymize traffic over the internet. For the purposes of a Bitcoin node, it also allows the user to make the node "publicly available", without having to set up the proper networking. This section of the guide will describe how to install and configure Tor for the Bitcoin node.

## Installing and configuring Tor

Now, install `tor`:

```
nala install -y tor
```

Open the `/etc/tor/torrc` file with a text editor:

```
sudo vim /etc/tor/torrc
```

Uncomment the following lines:

```
ControlPort 9051
CookieAuthentication 1
```

Below `CookieAuthentication`, add the following line:

```
CookieAuthFileGroupReadable 1
```

Exit the file, and reload the `tor` configurations with the following command:

```
systemctl reload tor
```

Confirm that `tor` is running with the following:

```
sudo ss -tulpn | grep tor | grep LISTEN
```

### Adding a hidden service

At any point, for any service, it is possible to create a hidden service for remote access (without a VPN connection). Examples of services that can be routed through Tor include:

- SSH
- Bitcoin Core
- Fulcrum
- Mempool.space

However, the Tor network is slower than other network traffic.

First, open the `/etc/tor/torrc` file with a text editor:

```
sudo vim /etc/tor/torrc
```

In the section marked as "just for location-hidden services", add the following four lines:

```
# Hidden Service for <SERVICE_NAME>
HiddenServiceDir /var/lib/tor/hidden_service_<SERVICE_NAME>
HiddenServiceVersion 3
HiddenServicePort <PORT_OF_SERVICE> 127.0.0.1:<PORT_OF_SERVICE>
```

Exit the file, and reload the `tor` configurations with the following command:

```
systemctl reload tor
```

Use `cat` to output the file contents in the corresponding directory to obtain the onion link to the service:

```
sudo cat /var/lib/tor/hidden_service_<SERVICE_NAME>/hostname
```

Now, it is possible to access that service remotely using Tor.

## References

- [Raspibolt - Privacy](https://raspibolt.org/guide/raspberry-pi/privacy.html)
    - Reference for initial setup of Tor for anonymization and hidden services.
