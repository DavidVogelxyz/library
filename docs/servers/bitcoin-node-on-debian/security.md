# Security

[Back to the home page](README.md)

## Table of contents

- [Configuring UFW](#configuring-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Other security settings](#other-security-settings)
- [References](#references)

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
sudo ufw allow from <IP_ADDRESS> proto tcp to any port 22
```

Add a firewall rule to allow HTTP connections from only specific IP addresses:

```
sudo ufw allow from <IP_ADDRESS> proto tcp to any port 80
```

Add a firewall rule to allow HTTPS connections from only specific IP addresses:

```
sudo ufw allow from <IP_ADDRESS> proto tcp to any port 443
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

## Other security settings

Next, install `fail2ban`, a package designed to temporarily ban users who fail to login too many times in a short period of time. No configuration is needed besides installing the package.

```
nala install -y fail2ban
```

Open the `/etc/security/limits.d/90-limits.conf` file:

```
sudo vim /etc/security/limits.d/90-limits.conf
```

To the end of the file, append the following lines:

```
*    soft nofile 128000
*    hard nofile 128000
root soft nofile 128000
root hard nofile 128000
```

Next, open the `/etc/pam.d/common-session` file:

```
sudo vim /etc/pam.d/common-session
```

To the end of the file, append the following line:

```
session required    pam_limits.so
```

Open the very similarly named `/etc/pam.d/common-session-noninteractive` file:

```
sudo vim /etc/pam.d/common-session-noninteractive
```

To the end of the file, append the same line as with the other PAM file:

```
session required    pam_limits.so
```

## References

- [Raspibolt - Security](https://raspibolt.org/guide/raspberry-pi/security.html)
    - Reference for securing the server using `ufw` and `fail2ban`.
