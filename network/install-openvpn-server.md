# Installing OpenVPN server

NB: This guide has been adapted from a [YouTube video](https://www.youtube.com/watch?v=Lk_v6Q0YsNo) by @MentalOutlaw.

## Introduction

This guide assists in the creation of new OpenVPN servers. OpenVPN is a free and open-source (FOSS) VPN server software that makes setting up a custom VPN extremely easy. To assist further, this guide will utilize an [OpenVPN install script](https://github.com/angristan/openvpn-install) by @Angristan. The script not only makes the server setup near instant and painless, but also streamlines the process of adding new user credentials and revoking any compromised credentials.

As is explained in other parts of the guide, these instructions can be deployed on anything from a cloud-hosted server to a virtual machine (VM) to a single-board computer like a Raspberry Pi.

## Table of Contents

- [Introduction](#introduction)
- [Initial setup](#initial-setup)
- [Setting up the virtual environment](#setting-up-the-virtual-environment)
- [Setting up SSH](#setting-up-ssh)
- [Setting up the OpenVPN server](#setting-up-the-openvpn-server)
- [References](#references)

## Initial setup

The first step in setting up an OpenVPN server is to create the virtual environment onto which OpenVPN will be installed. This virtual environment can take many forms, such as a cloud server, a Proxmox VM, or a standalone device such as a Raspberry Pi.

The following are suggestions for how to set up each of those options:

- Cloud server: spin up a Debian 12 cloud server using a cloud service provider (such as Vultr or Linode)
- Proxmox VM: spin up a Debian 12 VM
- Raspberry Pi: install Raspbian (Debian for Raspberry Pi) or Ubuntu Server (ARM version)

Once the virtual environment has been provisioned and initialized, the next step is to configure the server.

## Setting up the virtual environment

After booting into the server and signing in (likely as the root user), the first step is to confirm that all currently installed packages are up to date. If the server is running Debian or Ubuntu, this is achieved through the command:

```
apt update && apt upgrade
```

The next step is to create a new user (ex: openvpn) to administer and manage the OpenVPN server. It is highly advisable that the root user is not used for this purpose. Whether using Debian/Ubuntu or an Arch-based distribution, the commands to create a new user are the same. Follow up the new user creation with a password change and a change of shell (on Debian, the default shell is "sh"):

```
useradd -G sudo -m $USER

passwd $USER

chsh -s /bin/bash $USER
```

It is also worth checking the "/etc/hostname" and "/etc/hosts" files at this point. Confirm that the hostname is correct for the OpenVPN server, and confirm that it is found in "/etc/hosts" under a loopback address. If any adjustments were made to either of these files, it is advisable to restart the server after and then to log back in.

Finally, depending on the network's setup, it may also make sense to include a software firewall, such as `ufw`. For a publicly accessible server, such as a cloud server VPN, it is highly advisable to configure `ufw` such that the available ports are limited as much as possible. However, it is not necessary to open port 1194 (OpenVPN's standard port) into the `ufw` rules, as OpenVPN is configured to listen to receive inbound connections despite the port being blocked by the firewall.

## Setting up SSH

A crucial step in the setup process is to properly configure SSH login. There are two main components to this:

- disabling "root login via SSH"
- disabling "password-based authentication"

By disabling both of those options, the server will be limited to only accepting SSH connections from non-root users, and those users will not be able to login using a password alone. Therefore, a public/private keypair will need to be configured to allow the user to login safely.

Create a key on the machine accessing the VPN server by using the following command:

```
ssh-keygen -b rsa -t 4096
```

Note that the above command works both with Linux machines and Windows computers (through PowerShell). During keyfile generation, the command will prompt the user for a password. It is a acceptable practice to have the keyfile password be the same as the user's password on the VPN server.

Next, the key needs to be copied over to the VPN server and added to the list of keys authorized for SSH. On a Linux machine, this is most easily accomplished through the use of the following command:

```
ssh-copy-id -i /PATH/TO/THE/KEY.pub $USER@$HOST
```

The benefit of `ssh-copy-id` is that the key will automatically be entered into the '~/.ssh/authorized_keys' file for that particular user.

If `ssh-copy-id` is unavailable, the next best method to add the key to the 'authorized_keys' file is to use `scp` to securely copy the 'KEY.pub' file over to the VPN server. Then, the contents of that 'KEY.pub' file can be appended to the '~/.ssh/authorized_keys' file for that user.

From a Windows machine, neither `ssh-copy-id` not `scp` are readily available in PowerShell. Therefore, the simplest method for a Windows admin is to open up an SSH connection to the VPN server, open up the '~/.ssh/authorized_keys' in a preferred text editor, and paste the pubkey's content into the file.

Once the key is confirmed as resident within the '~/.ssh/authorized_keys' file, it is time to update the '/etc/ssh/sshd_config' file by disabling root access and password-based authentication. Since this is a file located in '/etc/...', open up the file using elevated privileges (eg. `sudo vim /etc/ssh/sshd_config`).

After opening the file, there are three lines of interest:

- PermitRootLogin
- PasswordAuthentication
- KbdInteractiveAuthentication

Whether by commenting out the already-existing lines and adding additional lines, or by replacing what's already there, those three lines should read:

```
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Once these lines have been edited, the `sshd` service requires a restart for the changes to take effect. This can be accompished with the following command:

```
sudo systemctl restart sshd
```

Once this command has been entered, test the changes by logging out and trying to log in again using the original `ssh $USER@$HOST` command. This should be refused with an error message of "Permission denied (publickey)." To connect using the keyfile that was added to the server, modify the command to read:

```
ssh -i /PATH/TO/THE/KEY.pub $USER@$HOST
```

This command should work, and will prompt the user for the password added during the key's creation.

Now that SSH logins have been secured, it's time to set up the OpenVPN server.

## Setting up the OpenVPN server

The easiest way to grab @Angristan's install script is to clone the git repo and work from there. First, confirm that the directory that will store the source files exists.

```
mkdir -pv ~/.local/src
```

Now that the directory is confirmed to exist, change directory into '~/.local/src'. Next, clone the git repo using the following command:

```
git clone https://github.com/angristan/openvpn-install.git
```

Change directory to the new 'openvpn-install' project and run the following command to start the script:

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
