SMTP server, running on Debian
==============================

NB: This guide has been adapted from a [YouTube video](https://www.youtube.com/watch?v=3dIVesHEAzc) by Luke Smith.

Introduction
------------

This guide is to assist a user with creating a simple webpage and SMTP server for a new domain, an example of which can be seen at [davidvogel.xyz](https://davidvogel.xyz). To assist further, this guide will utilize an [mail server install script](https://github.com/lukesmithxyz/emailwiz) by @lukesmithxyz. The script makes the server setup near instant and painless.

Table of contents
-----------------

- [Introduction](#introduction)
- [First steps](#first-steps)
- [Configuring DNS records](#configuring-dns-records)
- [Initial configuration](#initial-configuration)
- [Configuring UFW](#configuring-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Configuring nginx](#configuring-nginx)
- [Obtaining SSL certificates for the new domain](#obtaining-ssl-certificates-for-the-new-domain)
- [Setting up the mail server](#setting-up-the-mail-server)
- [Using the new mail server](#using-the-new-mail-server)
- [References](#references)

First steps
-----------

Before getting started, make sure to have a domain purchased and ready to go. This is easily done by going to the website of a registrar, such as [NameCheap](https://namecheap.com), and purchasing a domain there. For the remainder of the guide, the example domain will be `example.com`.

Next, make sure to have a cloud server spun up and available. This guide is written with a Debian server in mind. Because the server is intended to be an SMTP server, it will need the cloud service provider (CSP) to have port 25 open for the server. If that isn't already the case, reach out with a support ticket and ask the CSP to open port 25 for the server.

In addition, when provisioning a Debian server with a CSP, be sure to label (name) the server with the domain to be used. Some CSPs hardcode the label into the machine as its hostname -- to avoid any potential issues, label the server's hostname with the domain.

**Note: in a setting where the SMTP server is paired with an separate OpenVPN server, the OpenVPN server will also need port 25 open, so that mail can be sent from a client like Thunderbird, from a user on the VPN, to the SMTP server.**

Configuring DNS records
-----------------------

As the cloud server is provisioning, go ahead and update the DNS records for the domain. Delete all records besides the nameserver (NS) and start of authority (SOA) records. On a registar such as NameCheap, those records are maintained on a separate page of the site, so go ahead and delete as many of the pre-existing DNS records as possible.

Then, create the following A (IPv4) and AAAA (IPv6) records:

```
A           @       $SERVER_IPv4
A           *       $SERVER_IPv4
A           www     $SERVER_IPv4
AAAA        @       $SERVER_IPv6
AAAA        *       $SERVER_IPv6
AAAA        www     $SERVER_IPv6
```

**Note: with some registrars, the `@` will instead be an "empty field". If one does not work, use the other.**

Obviously, `$SERVER_IPv4` is the IPv4 address of the new cloud server, and `$SERVER_IPv6` is the IPv6 address of the server.

Then, create an MX record:

```
MX          @       mail.$DOMAIN.       10
```

`$DOMAIN` is the domain of the server. In the case of the example, the record would point to `mail.example.com.` In addition, the `10` is the priority of the record -- a lower value will always be preferred over the higher value.

**Note: the period after the domain is *intentional*. Most registrars will expect the period following the domain.**

Initial configuration
---------------------

Before signing in to the newly created server, it is also a good idea to configure reverse DNS (rDNS) records with the CSP. Find the relevant section of the CSP's server page, and make sure to add a rDNS (PTR) record for both the IPv4 and IPv6 addresses.

If the DNS records have already propagated, then it is possible to SSH using the domain, instead of the IP address. If the DNS records have not yet propagated, feel free to add a record to "/etc/hosts" of the client connecting to the server, in order to be able to SSH using the domain of the server.

Once logged in, follow this guide on [configuring a server running Debian](/servers/configuring-debian-server.md) to set up a new user account and secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

Configuring UFW
---------------

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

Add a firewall rule to allow SMTP connections:

```
sudo ufw allow stmp
```

Add a firewall rule to allow SMTPS connections:

```
sudo ufw allow stmps
```

Add a firewall rule to allow STARTTLS connections:

```
sudo ufw allow submission
```

Add a firewall rule to allow IMAPS connections:

```
sudo ufw allow imaps
```

Add a firewall rule to allow POP3 connections:

```
sudo ufw allow pop3
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

Add a firewall rule to allow SMTP connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 25
```

Add a firewall rule to allow SMTPS connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 465
```

Add a firewall rule to allow STARTTLS connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 587
```

Add a firewall rule to allow IMAPS connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 993
```

Add a firewall rule to allow POP3 connections from only specific IP addresses:

```
sudo ufw allow from $IP_ADDRESS proto tcp to any port 995
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

Configuring nginx
-----------------

Note: for this next section, references to `$HOSTNAME` represent the domain of the new server, without the TLD (eg. example; not example.com). In contrast, `$DOMAIN` represents the domain of the new server, *including* the TLD (eg. example.com).

First, update the package repositories, and install some packages that will be used throughout the SMTP server setup process:

```
nala update && nala install -y nginx python3-certbot-nginx
```

Now, create a new file at "/etc/nginx/sites-available/$HOSTNAME" and add the following content to it:

```
server {
    listen 80 ;
    listen [::]:80 ;

    root /var/www/$HOSTNAME ;

    index index.html index.htm index.nginx-debian.html;

    server_name $DOMAIN www.$DOMAIN ;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Obviously, change references to `$HOSTNAME` and `$DOMAIN` to point to the new domain. For the user's ease, `$HOSTNAME` is found under `root`, and `$DOMAIN` is found twice under `server_name`.

Next, create a new file at "/etc/nginx/sites-available/mail", and add the following content to it:

```
server {
    listen 80 ;
    listen [::]:80 ;

    root /var/www/mail ;

    index index.html index.htm index.nginx-debian.html;

    server_name mail.$DOMAIN www.mail.$DOMAIN ;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Again, change references to `$DOMAIN` to point to the new domain. For the user's ease, `$DOMAIN` is found twice under `server_name`.

Once both files have been configured, create symlinks to both in the "/etc/nginx/sites-enabled" directory.

First, the domain's webpage:

```
sudo ln -s /etc/nginx/sites-available/$HOSTNAME /etc/nginx/sites-enabled
```

Next, the mail server:

```
sudo ln -s /etc/nginx/sites-available/mail /etc/nginx/sites-enabled
```

With both symlinks created, restart the `nginx` service:

```
systemctl restart nginx
```

Now, create a directory within "/var/www":

```
sudo mkdir -pv /var/www/$HOSTNAME
```

Add the following content to the new file "/var/www/$HOSTNAME/index.html":

```
<h1>Thank you for visiting $DOMAIN!</h1>
```

Go to any web browser, and navigate to the new domain. The message "Thank you for visiting `$DOMAIN`!" should appear.

Obtaining SSL certificates for the new domain
---------------------------------------------

With the webpage set up, it is now time to obtain SSL certificates for the webpage. Since this will be a public website, generate SSL certificates signed by "Let's Encrypt".

To do this, run the following command:

```
sudo certbot --nginx
```

Enter a valid e-mail address, agree to the Terms of Service and choose whether or not to share that e-mail address with the Electronic Frontier Foundation (EFF).

Now, `certbot` will ask which names for which to activate HTTPS for. There should be four available options:

```
$DOMAIN
mail.$DOMAIN
www.$DOMAIN
www.mail.$DOMAIN
```

If there are not four options, something has been misconfigured. However, `certbot` can be run again in the future to generate SSL certificates for any missing names.

To add SSL for all names, just press "Enter".

Now, refresh the webpage, and the browser should be redirected to the HTTPS version of the site.

Setting up the mail server
--------------------------

The easiest way to install the mail server is to grab an install script written by @lukesmithxyz and deploy it.

The simplest way to do this is to clone the `git` repo and work from there. First, confirm that the directory that will store the source files exists.

```
mkdir -pv ~/.local/src
```

Now that the directory is confirmed to exist, change directory into '~/.local/src'. Next, clone the git repo using the following command:

```
git clone https://github.com/lukesmithxyz/emailwiz
```

Change directory to the new 'emailwiz' directory and run the following command to start the script:

```
sudo bash emailwiz.sh
```

Follow the on-screen prompts to set up the mail server. As a note, choose "Internet Site" for "Postfix Configuration", and enter "$DOMAIN" for the "System mail name" (eg. example.com).

The script should run without error. If the script returns with an error due to IPv6, there are a few different ways to remedy this. First, confirm that the `AAAA` DNS records have been added for the site. Next, create a hosts file entry ("/etc/hosts") for the IPv6 address. With both of these configured, re-run the `emailwiz.sh` script, and it should complete without any additional errors.

When the script completes successfully, there should be some output regarding records to be added to the site's DNS records. Create three new DNS records containing the output. As a note, the name for the DKIM key can usually be just `mail._domainkey`, and the name for the DMARC record can usually be just `_dmarc`.

Also, note that the SPF record may contain a reference to the localhost IPv4 record, rather than the server's public IPv4 address. Make sure to correct this if necessary.

Using the new mail server
-------------------------

With the mail server up and running, the first thing to do is to create a new mail user. Do this with the following command:

```
sudo useradd -G mail -m $USERNAME
```

Be sure to also set a password for this user with the following command:

```
sudo passwd $USERNAME
```

Now that a user has been created and a password has been set, use a mail client such as Thunderbird to attempt to login to the mail account. Using the default configurations, simply give the user a display name, and enter the username and password. This should be enough for Thunderbird to successfully sign in to the mail account. Be sure to select "IMAP" over "POP3" for the e-mail protocol.

Now that the mailbox has been accessed, attempt to send an e-mail to another mail account. If everything has been set up correctly, and the SMTP port (25) isn't blocked, the outbound e-mail should be immediately available in the recipient's inbox.

Congrats on setting up a new e-mail server and webpage!

References
----------

- [YouTube - Luke Smith - Setting up a Website and Email Server in One Sitting (Internet Landchad)](https://www.youtube.com/watch?v=3dIVesHEAzc)
- [GitHub - lukesmithxyz - emailwiz](https://github.com/lukesmithxyz/emailwiz)
