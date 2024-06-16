# DNS server using dnsmasq, running on Alpine Linux

NB: This guide has been tested using an Alpine Linux container on Proxmox, using the default Alpine image that Proxmox pulls.

## Table of contents

- [Initial configuration](#initial-configuration)
- [Configuring UFW](#configuring-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Installing dnsmasq](#installing-dnsmasq)
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

Add a firewall rule to allow DNS queries:

```
sudo ufw allow domain
```

### IP-specific firewall rules

Add a firewall rule to allow SSH connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 22
```

Add a firewall rule to allow DNS queries from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 53
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

## Installing dnsmasq

Now, update the package repositories, and install `dnsmasq` and `dnsmasq-openrc` to the Alpine server.

```
apk update && apk add dnsmasq dnsmasq-openrc
```

Next, using a text editor, add entries to the "/etc/hosts" file that the DNS server should resolve.

```
sudo vim /etc/hosts
```

Once some entries have been added to "/etc/hosts", the next step is to configure `dnsmasq`. Do this by editing the "/etc/dnsmasq.conf" file using a text editor.

```
sudo vim /etc/dnsmasq.conf
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
