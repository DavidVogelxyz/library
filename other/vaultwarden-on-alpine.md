# Vaultwarden password manager, running on Alpine Linux

NB: This guide has been tested using an Alpine Linux container on Proxmox, using the default Alpine image that Proxmox pulls.

NB: When using a tty on Alpine Linux, "Ctrl-Alt-Del" reboots the computer.

## Table of contents

- [Configure SSH](#Configure-SSH)
- [Add configuration files](#Add-configuration-files)
- [Install Vaultwarden](#Install-Vaultwarden)
- [Configure SSL and HTTPS connections](#Configure-SSL-and-HTTPS-connections)
- [Configure nginx](#Configure-nginx)
- [Final steps](#Final-steps)
- [Additional notes - How to change the admin token password](#Additional-notes---How-to-change-the-admin-token-password)
- [Additional notes - Adding SMTP to Vaultwarden](#Additional-notes---Adding-SMTP-to-Vaultwarden)

## Configure SSH

First, update the `apk` package list and install some packages that will be used throughout the Vaultwarden installation process.

```
apk update && apk add openssh-server vim git curl tmux nginx openssl
```

Next, configure `ssh` by editing the "/etc/ssh/sshd_config" file. Most notably, "PermitRootLogin" needs to be set as "yes", as the Alpine Linux container doesn't have a user created by default.

```
vim /etc/ssh/sshd_config
```

At least on the Alpine Linux container for Proxmox, when `ssh` is installed, `sshd` is not automatically configured to run as a service. This can be a problem on a container restart, if the service is expected to be running on reboot.

To ensure that `ssh` lockouts do not occur, run the following command:

```
rc-service sshd start && rc-update add sshd default
```

NB: On multiple occasions, this guide makes reference to the `rc-service` command. Know that `service` functions the same way as `rc-service`. In addition, `rc-update add $SERVICE` is a shorter way to write `rc-update add $SERVICE default`; both commands will add the "$SERVICE" to the "default" run level.

In order to test `ssh` login, get the IP address of the VM container.

```
ip a
```

Also, change the password (if necessary).

```
passwd
```

Especially if using the root user for configuration, it is best practice to use a SSH key for login. A key can be created on the `ssh` client computer by running the following:

```
ssh-keygen -t rsa -b 4096
```

Once the key is created, copy it over from the `ssh` client using the following command. Obviously, "$ALPINELINUX" should either be the hostname of the Alpine Linux server, or its IP address.

```
ssh-copy-id -i /PATH/TO/SSH/KEY root@$ALPINELINUX
```

At this point, the "PasswordAuthentication" setting in "/etc/ssh/sshd_config" should be explicity set to "no". The service can be started with:

```
service sshd restart
```

## Add configuration files

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
sed "s/PS1='\\\h/PS1='\\\u@\\\h/g" /etc/profile
```

## Install Vaultwarden

First, enable packages from the "edge" repository by echoing the following command into the "/etc/apk/repositories" file:

```
echo "https://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
```

Update the package list with the additions from the "edge" repository and install `vaultwarden` and `vaultwarden-web-vault`. The `vaultwarden-openrc` package is installed during the install of `vaultwarden`, so no need to include it.

```
apk update && apk add vaultwarden vaultwarden-web-vault
```

Next, add `vaultwarden` into the OpenRC default services and start it up.

```
rc-update add vaultwarden default && rc-service vaultwarden start
```

To confirm that the service is running correctly, attempt the following and assess the output:

```
curl localhost:8000
curl localhost:8000/admin
```

Neither page should load correctly.

`localhost:8000` should return a 404 error, and the `localhost:8000/admin` page should return a message about configuring an admin token.

Using a ***good password***, an admin token can be created with the following command:

```
vaultwarden hash
```

By editing either the "/etc/conf.d/vaultwarden" (default configurations) file, or the "/var/lib/vaultwarden/.env" (overriding changes) file, the admin token can be added in.

NB: While the ".env" file should override any and all changes made, issues seem to occur when using it as opposed to the "/etc/conf.d/vaultwarden" file. Therefore, it is suggested to use "/etc/conf.d/vaultwarden" instead of the ".env" file.

Open either file in an editor of choice:

```
vim /etc/conf.d/vaultwarden
```

or:

```
vim /var/lib/vaultwarden/.env
```

If editing "/etc/conf.d/vaultwarden", the following options should be set **using the `export` command**:

```
export ADMIN_TOKEN='$argon2id'
export DATA_FOLDER=/var/lib/vaultwarden
export DOMAIN=https://localhost
export ORG_EVENTS_ENABLED=true
export WEB_VAULT_ENABLED=true
export WEB_VAULT_FOLDER=/usr/share/webapps/vaultwarden-web/
```

If editing the "/var/lib/vaultwarden/.env" file, the following options can be set *without* using the `export` command:

```
ADMIN_TOKEN='$argon2id'
DATA_FOLDER=/var/lib/vaultwarden
DOMAIN=https://localhost
ORG_EVENTS_ENABLED=true
WEB_VAULT_ENABLED=true
WEB_VAULT_FOLDER=/usr/share/webapps/vaultwarden-web/
```

In either case, the ADMIN_TOKEN variable should reflect the admin token that was created with the `vaultwarden hash` command. Also, a DOMAIN of "https://localhost" will work to get the server running; however, if/when implementing SMTP, "DOMAIN" should reflect the domain on which Vaultwarden is being served (ex. "https://vaultwarden.domain.com").

Once the environment variables are set, the `vaultwarden` service should be restarted using the following command:

```
rc-service vaultwarden restart
```

With this configuration in place, the following curl commands should return different output than before:

```
curl localhost:8000
curl localhost:8000/admin
```

`localhost:8000` should return output for the web-vault login page, and the `localhost:8000/admin` page should return a page related to signing in to the admin panel.

## Configure SSL and HTTPS connections

Now, it's time to set up some self-signed SSL certificates in order to enable HTTPS connections. Create the following directory and change directory into it.

```
mkd /etc/nginx/ssl && cd /etc/nginx/ssl
```

Using the following two commands, create the required files.

Replace "[$SERVER]" with a name for the files being generated. This will likely be a name like "vaultwarden" or "server".

The first command creates the server's private key, as well as a "certificate signing request" (CSR) file.

```
openssl req -newkey rsa:4096 -nodes -keyout [$SERVER].key -out [$SERVER].csr
```

The next command uses the private key and the CSR to generate a certificate file.

```
openssl x509 -signkey [$SERVER].key -in [$SERVER].csr -req -days 36500 -out [$SERVER].crt
```

## Configure nginx

Create a new configuration file in the "/etc/nginx/http.d" directory:

```
vim /etc/nginx/http.d/vaultwarden.conf
```

Configure the file to look something similar to the below configuration file.

Obviously, swap out "[$HOSTNAME]" with the hostname of the server running `vaultwarden`, and change "[$SERVER]" to the name of the '.crt' and '.key' files that were created in the previous step.

```
server {
    listen 80;
    listen [::]:80;
    return 301 https://[$HOSTNAME]$request_uri;

    server_name [$HOSTNAME];
}

server {
    listen 443 ssl;
    listen [::]:443 ;

    server_name [$HOSTNAME];

    ssl_certificate /etc/nginx/ssl/[$SERVER].crt;
    ssl_certificate_key /etc/nginx/ssl/[$SERVER].key;

    location / {
        proxy_pass http://localhost:8000;
    }
}
```

With the configuration file set up, add the `nginx` service to the list of default services. Then, start the service, and curl the output.

```
rc-update add nginx default && rc-service nginx start
```

To test the `nginx` configuration, curl the following (obviously, change "[$HOSTNAME]" to the hostname of the server):

```
curl https://[$HOSTNAME]
```

This should display the same output as curling "localhost:8000".

## Final steps

With all of this configuration in place, accessing the server at https://[$HOSTNAME] should return the Vaultwarden login page. Also, accessing https://[$HOSTNAME]/admin should return the administration panel. Simply enter the password that was used to generate the admin token to explore the admin panel.

Congrats on setting up a Vaultwarden instance using Alpine Linux!

## Additional notes - How to change the admin token password

In order to change the admin token for the Vaultwarden admin panel, settings must be changed in the correct files. If both files exist, both "/etc/conf.d/vaultwarden" ***AND*** "/var/lib/vaultwarden/config.json" must be edited for the changes to occur.

As before, "/etc/conf.d/vaultwarden" needs to have its `export ADMIN_TOKEN='$argon2id'` changed to the new token. However, in addition, be sure to check for a "/var/lib/vaultwarden/config.json" file. It appears that this file is only created after settings are changed via the Vaultwarden admin panel webpage. If it exists, be sure to change the "ADMIN_TOKEN" value here as well.

Note that this is also true for values such as "DOMAIN" or "SMTP_FROM_NAME" -- the setting needs to be changed in "/etc/conf.d/vaultwarden" **AND** in "/var/lib/vaultwarden/config.json" (if the file exists).

## Additional notes - Adding SMTP to Vaultwarden

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
