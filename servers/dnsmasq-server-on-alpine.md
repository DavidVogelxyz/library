# DNS server using dnsmasq, running on Alpine Linux

NB:

- This guide has been tested using an Alpine Linux container on Proxmox, using the default Alpine image that Proxmox pulls.
- On multiple occasions, this guide makes reference to the `service` command. Know that `service` functions the same way as `rc-service`.
- In addition, `rc-update add $SERVICE` is a shorter way to write `rc-update add $SERVICE default`.
    - Both commands will add the `$SERVICE` to the "default" run level.
- When using a tty on Alpine Linux, "Ctrl-Alt-Del" reboots the computer.

## Table of contents

- [Configuring SSH](#configuring-ssh)
    - [On-prem Proxmox install](#on-prem-proxmox-install)
    - [Securing SSH](#securing-ssh)
- [Adding configuration files](#adding-configuration-files)
- [Installing UFW](#installing-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Installing dnsmasq](#installing-dnsmasq)
- [References](#references)

## Configuring SSH

First, update the `apk` package list and install some packages that will be used throughout the DNS server installation process.

```
apk update && apk add openssh-server vim tmux ufw
```

At this point, the guide splits depending on whether the install is local / on-premises (on-prem) or via a cloud service provider. For an on-prem solution, start at [on-prem Proxmox install](#on-prem-proxmox-install); for a cloud service provider, start at [securing SSH](#securing-ssh).

### On-prem Proxmox install

Configure `ssh` by editing the "/etc/ssh/sshd_config" file. Most notably, "PermitRootLogin" needs to be set as "yes", as the Alpine Linux container doesn't have a user created by default.

```
vim /etc/ssh/sshd_config
```

At least on the Alpine Linux container for Proxmox, when `ssh` is installed, `sshd` is not automatically configured to run as a service. This can be a problem on a container restart, if the service is expected to be running on reboot.

To ensure that `ssh` lockouts do not occur, run the following command:

```
service sshd start && rc-update add sshd
```

In order to test `ssh` login, get the IP address of the VM container.

```
ip a
```

### Securing SSH

Especially when using the root user for configuration, it is best practice to use a SSH key for login. A key can be created on the `ssh` client computer by running the following:

```
ssh-keygen -t rsa -b 4096
```

Once the key is created, copy it over from the `ssh` client using the following command. Obviously, "$ALPINELINUX" should either be the hostname of the Alpine Linux server, or its IP address.

```
ssh-copy-id -i /PATH/TO/SSH/KEY root@$ALPINELINUX
```

Edit the "/etc/ssh/sshd_config" file and secure `ssh` by updating the following:

- "PermitRootLogin" should be set to "prohibit-password".
- "PasswordAuthentication" should be explicity set to "no".

The service can be restarted with:

```
service sshd restart
```

Also, change the password (if necessary).

```
passwd
```

In addition, for cloud install, the "/etc/hostname" and "/etc/hosts" files may need to be updated to reflect the correct hostname for the server.

## Adding configuration files

First, create some directories that will store some of the configuration files:

```
mkdir -pv ~/.local/src ~/.config/shell
```

Change directory into the newly created "~/.local/src" directory.

```
cd ~/.local/src/
```

Clone a few GitHub repositories with already-created configuration files.

```
git clone https://github.com/davidvogelxyz/dotfiles && git clone https://github.com/davidvogelxyz/vim
```

Return to the home directory of the active user.

```
cd
```

Symbolically link the `vim` configurations to the home directory of the active user.

```
ln -s ~/.local/src/vim ~/.vim
```

Copy the "aliasrc" file for Alpine Linux into the newly created "~/.config/shell" directory.

```
cp ~/.local/src/dotfiles/.config/shell/aliasrc-alpine ~/.config/shell/aliasrc && source ~/.config/shell/aliasrc
```

To the "~/.ashrc" file, add the line `source ~/.config/shell/aliasrc`. To the "/etc/profile" file, add the line `source ~/.ashrc`.

```
echo "source ~/.config/shell/aliasrc" >> ~/.ashrc && echo -e "\nsource ~/.ashrc" >> /etc/profile
```

Another change that, while minimal, can be helpful, is to change the "/etc/profile" file so that the username shows along with the hostname. This can easily be accomplished with the following `sed` command:

```
sed -i "s/PS1='\\\h/PS1='\\\u@\\\h/g" /etc/profile
```

## Installing UFW

Add a default firewall rule to deny incoming connections:

```
ufw default deny incoming
```

Add a default firewall rule to allow outgoing connections:

```
ufw default allow outgoing
```

Set logging to off:

```
ufw logging off
```

For generalized firewall rules, see [general firewall rules](#general-firewall-rules); to create firewall rules that only allow a certain IP to access the services, see [IP-specific firewall rules](#ip-specific-firewall-rules).

### General firewall rules

Add a firewall rule to allow SSH connections:

```
ufw allow ssh
```

Add a firewall rule to allow DNS queries:

```
ufw allow domain
```

### IP-specific firewall rules

Add a firewall rule to allow SSH connections from only specific IP addresses:

```
ufw allow from $IP_ADDRESS proto tcp to any port 22
```

Add a firewall rule to allow DNS queries from only specific IP addresses:

```
ufw allow from $IP_ADDRESS proto tcp to any port 53
```

### Enabling UFW

Enable `ufw`:

```
ufw enable
```

Enable `ufw` to run on startup:

```
rc-update add ufw
```

## Installing dnsmasq

First, begin by installing `dnsmasq` and `dnsmasq-openrc` to the Alpine server.

```
apk add dnsmasq dnsmasq-openrc
```

Next, using a text editor, add entries to the "/etc/hosts" file that the DNS server should resolve.

```
vim /etc/hosts
```

Once some entries have been added to "/etc/hosts", the next step is to configure `dnsmasq`. Do this by editing the "/etc/dnsmasq.conf" file using a text editor.

```
vim /etc/dnsmasq.conf
```

The esssential edits are as follows:

- Uncomment `#domain-needed`.
    - By adding `domain-needed` to the configuration file, `dnsmasq` knows not to send hostname to the upstream DNS server if the hostname fails to resolve.
    - As an example, consider a machine named "example.localdomain". If the hostname "example" was queried, but failed to resolve, the DNS server would not pass that hostname to the upstream DNS server.
        - However, a query of "example.localdomain" would forward to the upstream DNS server, as it has a domain.
- Uncomment `#bogus-priv`.
    - By adding `bogus-priv` to the configuration file, `dnsmasq` knows not to send a local IP to the upstream DNS server if the IP fails its reverse-lookup query.
    - As an example, an IP address of 192.168.4.13 would not pass to the upstream DNS server if the reverse-lookup query failed to return.
- Add upstream DNS servers with `server=$IP_address`.
    - As an example, to add Quad9 (9.9.9.9) as an upstream server, add the line `server=9.9.9.9` to the configuration file.
- Increase the cache size of the DNS server by changing `cache-size=$CACHE_SIZE`.
    - The default for `cache-size` is only 150 entries.
    - To reduce query latency, increase this number to a more reasonable number.
        - A "reasonable" number is entirely dependent on what sorts of queries the server will be handling, the system resources allocated to the server, and the amount of clients the DNS server will be servicing.

Once the edits have been made, start the service with the following command:

```
service dnsmasq start
```

Once the service has been started, confirm that `dnsmasq` is running with the following command:

```
service dnsmasq status
```

Also, be sure to add `dnsmasq` to the default run level, so that the service will start up whenever the server boots up.

```
rc-update add dnsmasq
```

To confirm the server is working, add the IP address of the DNS server to the DNS server list of a second computer. If using a Linux machine, do this by editing "/etc/resolv.conf" and adding `nameserver $IP_ADDRESS` to the top of the list of nameservers. If the server is working, pinging a domain that has an entry in the DNS server's "/etc/hosts" file should result in resolution of the domain name, and a successful ping.

## References

- [How to Geek - How to Run Your Own DNS Server on Your Local Network](https://www.howtogeek.com/devops/how-to-run-your-own-dns-server-on-your-local-network/)
