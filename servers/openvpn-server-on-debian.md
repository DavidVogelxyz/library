# OpenVPN server, running on Debian

NB: This guide has been adapted from a [YouTube video](https://www.youtube.com/watch?v=Lk_v6Q0YsNo) by @MentalOutlaw.

## Introduction

This guide assists in the creation of new OpenVPN servers. OpenVPN is a free and open-source (FOSS) VPN server software that makes setting up a custom VPN extremely easy. To assist further, this guide will utilize an [OpenVPN install script](https://github.com/angristan/openvpn-install) by @Angristan. The script not only makes the server setup near instant and painless, but also streamlines the process of adding new user credentials and revoking any compromised credentials.

As is explained in other parts of the guide, these instructions can be deployed on anything from a cloud-hosted server to a virtual machine (VM) to a single-board computer like a Raspberry Pi.

## Table of Contents

- [Introduction](#introduction)
- [Setting up a cloud server](#setting-up-a-cloud-server)
- [Initial configuration](#initial-configuration)
- [Configuring UFW](#configuring-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Setting up the OpenVPN server](#setting-up-the-openvpn-server)
- [References](#references)

## Setting up a cloud server

The first step in setting up an OpenVPN server is to create the virtual environment onto which OpenVPN will be installed. This virtual environment can take many forms, such as a cloud server, a Proxmox VM, or a standalone device such as a Raspberry Pi.

The following are suggestions for how to set up each of those options:

- Cloud server: spin up a Debian 12 cloud server using a cloud service provider (such as Vultr or Linode)
- Proxmox VM: spin up a Debian 12 VM; reference [guide on installing Debian](/install-os/install-debian.md) for assistance with this.
- Raspberry Pi: install Raspbian (Debian for Raspberry Pi) or Ubuntu Server (ARM version)

Once the virtual environment has been provisioned and initialized, the next step is to configure the server.

## Initial configuration

Follow this guide on [configuring a server running Debian](/servers/configuring-debian-server.md) to set up a new user account and secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

## Configuring UFW

Depending on the network's setup, it may also make sense to include a software firewall, such as `ufw`. For a publicly accessible server, it is highly advisable to configure `ufw` such that the available ports are limited as much as possible.

It does not seem to be necessary to open port 1194 (OpenVPN's standard port) on the `ufw` rules, as OpenVPN is configured to listen to receive inbound connections despite the port being blocked by the firewall. Despite this, the guide describes how to open the OpenVPN port when using `ufw`.

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

Add a firewall rule to allow OpenVPN connections:

```
sudo ufw allow openvpn
```

### IP-specific firewall rules

Add a firewall rule to allow SSH connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 22
```

Add a firewall rule to allow OpenVPN connections from only specific IP addresses (this is not advised, as it will limit the utility of the VPN):

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 1194
```

### Enabling UFW

Enable `ufw`:

```
sudo ufw enable
```

Enable `ufw` to run on startup:

```
systemctl enable ufw
```

To check and confirm the firewall rules, use the following command:

```
sudo ufw status
```

## Setting up the OpenVPN server

The easiest way to install an OpenVPN server is to grab an install script written by @Angristan and deploy it.

The simplest way to do this is to clone the `git` repo and work from there. First, confirm that the directory that will store the source files exists.

```
mkdir -pv ~/.local/src
```

Now that the directory is confirmed to exist, change directory into '~/.local/src'. Next, clone the git repo using the following command:

```
git clone https://github.com/angristan/openvpn-install
```

Change directory to the new 'openvpn-install' directory and run the following command to start the script:

```
sudo bash openvpn-install.sh
```

Follow the on-screen prompts to set up the VPN server. The default configurations should be more than fine for most use cases: IPv6 support is unnecessary; port 1194 for OpenVPN is fine; AdGuard DNS (option 11) is a good choice; no need to enable compression or encryption. And, that's it. Let the script run, and it will soon ask for a client name. This client name is the name for the credentials for any particular one user.

Remember: credentials can only be used by one user at a time. So, if there are going to be multiple connections to the VPN at the same time, each user will need their own credentials. While password are not required for user credentials, they are suggested. Otherwise, anyone who gains access to the 'FILE.ovpn' can access the VPN.

The last step is to recover the client's configuration file from the server so that it can be used for VPN connections. Whether using a Linux or Windows machine, the command is the same:

```
sftp -i /PATH/TO/THE/KEY.pub $USER@$HOST
```

The client files should be dumped into the user's home folder, so the only command that *should* be needed from within `sftp` is the `get FILE.ovpn` command.

With this configuration file, the user should be able to import the configuration file into their OpenVPN client and connect to the server!

## References

- [GitHub - Angristan - openvpn-install](https://github.com/angristan/openvpn-install)
- [YouTube - MentalOutlaw - How to Create Your Own VPN (and why)](https://www.youtube.com/watch?v=Lk_v6Q0YsNo)
