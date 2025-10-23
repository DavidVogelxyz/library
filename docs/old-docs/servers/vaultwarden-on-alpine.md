Vaultwarden password manager, running on Alpine Linux
=====================================================

NB:

- This guide has been tested using an Alpine Linux container on Proxmox, using the default Alpine image that Proxmox pulls. In addition, this guide has also been tested using VMs hosted on Vultr and Linode.
- This guide makes frequent references to the IP address `127.0.0.1`. Obviously, this is a reference to the hostname `localhost`. Either can be used in place of the other; however, using the IP address itself avoids both the "host file lookup", as well as the potential for trying the IPv6 entry for `localhost`.

Table of contents
-----------------

- [Initial configuration](#initial-configuration)
- [Configuring UFW](#configuring-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Installing Vaultwarden](#installing-vaultwarden)
    - [Securing the Vaultwarden admin panel](#securing-the-vaultwarden-admin-panel)
- [Configuring SSL and HTTPS connections](#configuring-ssl-and-https-connections)
- [Configuring nginx](#configuring-nginx)
- [Final steps](#final-steps)
- [Additional notes](#additional-notes)
    - [Changing the admin token password](#changing-the-admin-token-password)
    - [Adding SMTP to Vaultwarden](#adding-smtp-to-vaultwarden)
    - [Webpage settings](#webpage-settings)
    - [Browser extensions](#browser-extensions)
    - [Fixing icons](#fixing-icons)
    - [Disabling new user registration](#disabling-new-user-registration)
- [References](#references)

Initial configuration
---------------------

First, update the `apk` package list and install some packages that will be used throughout the Vaultwarden server setup process.

```
apk update && apk add curl git openssh-server sudo tmux ufw vim
```

Then, enable packages from the "edge" repository by echoing the following command into the "/etc/apk/repositories" file:

```
echo "https://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
```

Next, follow this guide on [configuring a server running Alpine Linux](/servers/configuring-alpine-server.md) to set up a new user account and secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

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

Installing Vaultwarden
----------------------

Now, update the package list with the additions from the "edge" repository, and install `vaultwarden` and `vaultwarden-web-vault`. The `vaultwarden-openrc` package is installed during the install of `vaultwarden`, so no need to include it. Also, install `nginx` and `openssl` at this time.

```
apk update && apk add vaultwarden vaultwarden-web-vault nginx openssl
```

Next, add `vaultwarden` into the OpenRC default services and start it up.

```
rc-update add vaultwarden && service vaultwarden start
```

To test that `vaultwarden` is running, `curl` the following:

```
curl 127.0.0.1:8000
```

Using `curl` on `127.0.0.1:8000` should return a 404 error.

Now, `curl` the following:

```
curl 127.0.0.1:8000/admin
```

Using `curl` on `127.0.0.1:8000/admin` should return a message about configuring an admin token.

### Securing the Vaultwarden admin panel

Using a ***good password***, an admin token can be created with the following command:

```
sudo vaultwarden hash
```

By editing either the "/etc/conf.d/vaultwarden" (default configurations) file, or the "/var/lib/vaultwarden/.env" (overriding changes) file, the admin token can be added in.

NB: While the ".env" file should override any and all changes made, issues seem to occur when using it as opposed to the "/etc/conf.d/vaultwarden" file. Therefore, it is suggested to use "/etc/conf.d/vaultwarden" instead of the ".env" file.

Open either file in an editor of choice:

```
sudo vim /etc/conf.d/vaultwarden
```

or:

```
sudo vim /var/lib/vaultwarden/.env
```

If editing "/etc/conf.d/vaultwarden", the following options should be set **using the `export` command**:

```
export ADMIN_TOKEN='$argon2id'
export DATA_FOLDER=/var/lib/vaultwarden
export DOMAIN=https://DOMAIN.TLD
export IP_HEADER=X-Forwarded-For
export ORG_EVENTS_ENABLED=true
export WEB_VAULT_ENABLED=true
export WEB_VAULT_FOLDER=/usr/share/webapps/vaultwarden-web/
```

If editing the "/var/lib/vaultwarden/.env" file, the following options can be set *without* using the `export` command:

```
ADMIN_TOKEN='$argon2id'
DATA_FOLDER=/var/lib/vaultwarden
DOMAIN=https://DOMAIN.TLD
IP_HEADER=X-Forwarded-For
ORG_EVENTS_ENABLED=true
WEB_VAULT_ENABLED=true
WEB_VAULT_FOLDER=/usr/share/webapps/vaultwarden-web/
```

In either case:

- The value for ADMIN_TOKEN should reflect the admin token that was created with the command: `vaultwarden hash`.
- A value for DOMAIN of "https://127.0.0.1" will work to get the server running.
    - However, if/when implementing SMTP, "DOMAIN" should reflect the domain on which Vaultwarden is being served (ex. "https://DOMAIN.TLD").
        - The importance of this setting can be seen when using SMTP with Vaultwarden.
            - When a new user registers with the server, the value for DOMAIN will be what appears in the e-mail.
                - "Thank you for creating an account at https://DOMAIN.TLD" vs "Thank you for creating an account at https://127.0.0.1".
- Setting the value for IP_HEADER as "X-Forwarded-For" will avoid an issue that can occur when implementing SMTP with Vaultwarden.
    - If "IP_HEADER" is left as the default (which seems to be "X-Real-IP"), then e-mails sent by Vaultwarden regarding "New Device Login" will only show the IP address of 127.0.0.1.
        - This is technically correct, as this is the IP address of the local `nginx` instance that is forwarding the requests over to the local Vaultwarden server.
        - However, this is unhelpful, as the purpose of reporting an IP address in a "New Device Login" e-mail would be to record the IP address of the actual user.
        - Therefore, setting the IP_HEADER as "X-Forwarded-For" will direct the server to use the forwarded IP address, which is the address for the user signing in.
            - In order for this to work correctly, another setting has to exist in the `nginx` configuration file. See the [`nginx` configuration section](#configuring-nginx) for more details.

Once the environment variables are set, the `vaultwarden` service should be restarted using the following command:

```
service vaultwarden restart
```

With this configuration in place, the following curl commands should return different output than before:

```
curl 127.0.0.1:8000
curl 127.0.0.1:8000/admin
```

`127.0.0.1:8000` should return output for the web-vault login page, and the `127.0.0.1:8000/admin` page should return a page related to signing in to the admin panel.

Configuring SSL and HTTPS connections
-------------------------------------

Now, it's time to set up some self-signed SSL certificates in order to enable HTTPS connections. Create the following directory and change directory into it.

```
sudo mkdir -pv /etc/nginx/ssl && cd /etc/nginx/ssl
```

Using the following two commands, create the required files.

Replace `$SERVER` with a name for the files being generated. This will likely be a name like "vaultwarden" or "server".

The first command creates the server's private key, as well as a "certificate signing request" (CSR) file.

```
sudo openssl req -newkey rsa:4096 -nodes -keyout $SERVER.key -out $SERVER.csr
```

The next command uses the private key and the CSR to generate a certificate file.

```
sudo openssl x509 -signkey $SERVER.key -in $SERVER.csr -req -days 36500 -out $SERVER.crt
```

Configuring nginx
-----------------

Create a new configuration file in the "/etc/nginx/http.d" directory:

```
sudo vim /etc/nginx/http.d/vaultwarden.conf
```

Configure the file to look something similar to the below configuration file.

Obviously, swap out `$HOSTNAME` with the hostname of the server running `vaultwarden`, and change `$SERVER` to the name of the '.crt' and '.key' files that were created in the previous step.

```
server {
    listen 80;
    listen [::]:80;
    return 301 https://$HOSTNAME$request_uri;

    server_name $HOSTNAME;
}

server {
    listen 443 ssl;
    listen [::]:443 ;

    server_name $HOSTNAME;

    ssl_certificate /etc/nginx/ssl/$SERVER.crt;
    ssl_certificate_key /etc/nginx/ssl/$SERVER.key;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

With the configuration file set up, add the `nginx` service to the list of default services. Then, start the service, and curl the output.

```
rc-update add nginx && service nginx start
```

To test the `nginx` configuration, curl the following (obviously, change `$HOSTNAME` to the hostname of the server):

```
curl https://$HOSTNAME
```

This should display the same output as curling "127.0.0.1:8000".

Final steps
-----------

With all of this configuration in place, accessing the server at https://$HOSTNAME should return the Vaultwarden login page. Also, accessing https://$HOSTNAME/admin should return the administration panel. Simply enter the password that was used to generate the admin token to explore the admin panel.

Congrats on setting up a Vaultwarden instance using Alpine Linux!

Additional notes
----------------

This section contains additional comments and troubleshooting steps that have been useful in administrating a Vaultwarden server.

### Changing the admin token password

In order to change the admin token for the Vaultwarden admin panel, settings must be changed in the correct files. If both files exist, both "/etc/conf.d/vaultwarden" ***AND*** "/var/lib/vaultwarden/config.json" must be edited for the changes to occur.

As before, "/etc/conf.d/vaultwarden" needs to have its `export ADMIN_TOKEN='$argon2id'` changed to the new token. However, in addition, be sure to check for a "/var/lib/vaultwarden/config.json" file. It appears that this file is only created after settings are changed via the Vaultwarden admin panel webpage. If it exists, be sure to change the "ADMIN_TOKEN" value here as well.

Note that this is also true for values such as "DOMAIN" or "SMTP_FROM_NAME" -- the setting needs to be changed in "/etc/conf.d/vaultwarden" **AND** in "/var/lib/vaultwarden/config.json" (if the file exists).

### Adding SMTP to Vaultwarden

To add SMTP (Simple Mail Transfer Protocol) functionality to the Vaultwarden server, simply uncomment and adjust the following settings in "/etc/conf.d/vaultwarden":

```
export SMTP_HOST=smtp.domain.tld
export SMTP_FROM=vaultwarden@domain.tld
export SMTP_FROM_NAME=Vaultwarden
export SMTP_SECURITY=starttls
export SMTP_PORT=587
export SMTP_USERNAME=username
export SMTP_PASSWORD=password
export SMTP_TIMEOUT=15
```

This is relatively self-explanatory, so there's no need to go in depth here.

However, as mentioned earlier, if "/var/lib/vaultwarden/config.json" exists, the "SMTP_FROM_NAME" will also need to be updated there.

### Webpage settings

Once logged in, there are a few settings that can (and should) be changed.

Navigate to "Settings > Preferences" and observe the default settings for "Vault timeout" (15 minutes) and "Theme" (light).

For most users, a "Vault timeout" of 15 minutes may be too high, and can be lowered to a setting that is convenient, yet secure. Also, most users will likely prefer the dark theme to the light theme.

### Browser extensions

Similarly to the webpage settings, there are a few settings that should be changed for the browser extension. Note that, as of 2024 August 8, both Chrome-based browsers and Firefox appear to have identical locations for their settings.

First, navigate to "Settings > Account Security" and observe the default setting for "Vault timeout" (on browser restart). Users may prefer to set this to a discrete timeout, rather than leaving their Vaultwarden unlocked for the duration of their browser session.

Next, navigate to "Settings > Auto-fill" and observe the default setting for "Clear clipboard" (never). It is probably a good security practice to have this set to a convenient (yet secure) value, such as 10 or 20 seconds.

Last, navigate to "Settings > Vault > Sync". While this isn't a setting, it is good to note that manual sync can always be achieved through this menu option.

- Note: for users that don't have a root certificate authority installed at the browser or OS level, there may be times when a sync failed with an error of "failed to fetch".
    - This is often due to the fact that the certificate needs to be (re)accepted.
    - Simply navigate to the URL of the Vaultwarden server, accept the security warning regarding the certificate, and attempt to sync again.
        - The user does **NOT** need to sign in on the webpage -- accepting the security warning so the page can load is sufficient to resolve this issue.

### Fixing icons

Sometimes, icons that previously displayed properly will fail to load. I experienced this when migrating a Vaultwarden Alpine container from one Proxmox instance to another. However, I am unsure whether this was due to the container being redeployed on a different Proxmox, or a different network. It didn't happen for all favicons; but, there were a certain handful that failed to load on the new server.

Regardless, there was a very simple way to resolve this issue.

The trick was to remove all icon files found in "/var/lib/vaultwarden/icon-cache". Once the icon files have been deleted, the user experiencing the issue should disable icons in the user's preferences (within the web vault's UI) and save the change. Then, re-enable icons and save again. This will force the server to query the URLs for icons, and all the icons will be downloaded again.

NB: One way to confirm whether the icons are loading is to go to "https://vaultwarden-domain.tld/icons/icon-domain.tld/icon.png" and see whether the icon loads correctly or not.

### Disabling new user registration

There may be times when the server should have "new user registration" off by default. An example of this is when a server is sufficiently public that there are concerns of malicious users creating (or, spamming) new accounts on the server.

This can be easily achieved by adding the following line to the "/etc/conf.d/vaultwarden" file:

```
export SIGNUPS_ALLOWED=false
```

Note that this only disables the ability for new users to register themselves using the "New around here? Create account" option on the sign-in page. By default, users can still be added by administrators in the admin panel through an e-mail invitation. This can further be restricted by adding the following line to the "/etc/conf.d/vaultwarden" file:

```
export INVITATIONS_ALLOWED=false
```

At this point, unless one of these two options are changed back to true (or, simply commented out), no new users can be added to the Vaultwarden server.

References
----------

- [Vaultwarden wiki](https://github.com/dani-garcia/vaultwarden/wiki)
- [YouTube - anthonywritescode - don't use localhost (intermediate) anthony explains #534](https://www.youtube.com/watch?v=98SYTvNw1kw)
    - A video referencing the benefits of using `127.0.0.1`, as opposed to `localhost`
- [GitHub - Vaultwarden - IP Address always reports as 127.0.0.1 in email logs](https://github.com/dani-garcia/vaultwarden/discussions/3485)
    - A GitHub issue that resolved the IP address discrepancy in the e-mail logs
- [GitHub - Vaultwarden - Proxy Examples](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples)
    - Documentation for Vaultwarden that goes into more details about how to configure `nginx` re: IP address discrepancies in e-mail logs (and other points about proxies)
