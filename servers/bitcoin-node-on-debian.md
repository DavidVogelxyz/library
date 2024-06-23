# Bitcoin node, running on Debian

## Introduction

The purpose and scope of this guide is to assist in the creation and configuration of a Bitcoin node.

A Bitcoin node is a server that providers to its users the ability to check transactions and review block history. While a Bitcoin node running Bitcoin Core is useful on its own, its value is magnified when using additional services and software that leverage a locally hosted timechain.

This guide has been written to simplify the installation of a Bitcoin node running Debian. In addition, this guide assumes installation on a dedicated computer; specifically, a laptop. As of the creation of this guide (2024 May 21), this entire installation requires 820GB of storage space, with Bitcoin Core requiring at minimum 50GB annually. This average only considers Bitcoin Core, not the size requirement of the transaction indexer, as is based on the assumption that the Bitcoin network appends 50,000 blocks a year, of a size of 1MB each. These days, this is a under-estimation. However, the calculation justifies the suggestion that a Bitcoin node should be created with *at minimum* 2TB of storage space.

## Table of contents

- [Introduction](#introduction)
- [First steps](#first-steps)
- [Initial configuration](#initial-configuration)
    - [Preventing laptop lid closes from suspending processes](#preventing-laptop-lid-closes-from-suspending-processes)
    - [Configuring zsh](#configuring-zsh)
    - [Finalizing initial configuration](#finalizing-initial-configuration)
- [Configuring UFW](#configuring-ufw)
    - [General firewall rules](#general-firewall-rules)
    - [IP-specific firewall rules](#ip-specific-firewall-rules)
    - [Enabling UFW](#enabling-ufw)
- [Other security settings](#other-security-settings)
- [Configuring nginx](#configuring-nginx)
- [Configuring Tor](#configuring-tor)
    - [Adding a hidden service](#adding-a-hidden-service)
- [Installing Bitcoin Core](#installing-bitcoin-core)
    - [Configuring Bitcoin Core](#configuring-bitcoin-core)
    - [Checking on Bitcoin Core during the intial sync](#checking-on-bitcoin-core-during-the-intial-sync)
    - [Re-configuring Bitcoin Core after the initial sync](#re-configuring-bitcoin-core-after-the-initial-sync)
- [Installing Fulcrum (Bitcoin transaction indexer)](#installing-fulcrum-bitcoin-transaction-indexer)
    - [Configuring zram-swap](#configuring-zram-swap)
    - [Configuring Fulcrum](#configuring-fulcrum)
    - [Re-configuring Fulcrum after the initial sync](#re-configuring-fulcrum-after-the-initial-sync)
    - [Adding a hidden service for Fulcrum](#adding-a-hidden-service-for-fulcrum)
    - [Editing the banner for Fulcrum](#editing-the-banner-for-fulcrum)
- [Installing Mempool.space (Timechain explorer)](#installing-mempool-space-timechain-explorer)
    - [Configuring Mempool](#configuring-mempool)
        - [Configuring the Mempool backend](#configuring-the-mempool-backend)
        - [Configuring the Mempool frontend](#configuring-the-mempool-frontend)
        - [Finalizing the Mempool configuration](#finalizing-the-mempool-configuration)
    - [Adding a hidden service for Mempool](#adding-a-hidden-service-for-mempool)
- [References](#references)

## First steps

The first step to creating a Bitcoin node would be to install Debian onto the computer that will be used as the node. To assist with this setup process, please refer to [this guide](https://github.com/DavidVogelxyz/library/blob/master/install-os/install-debian.md) that details how to install Debian. Also, please note that [this "debian-setup" script](https://github.com/DavidVogelxyz/debian-setup) will help to speed up the process.

## Initial configuration

Once logged in, follow this guide on [configuring a server running Debian](/servers/configuring-debian-server.md) to secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

Note: since it's likely that the "install Debian" guide was used to set up the node, then a user should have already been created.

### Preventing laptop lid closes from suspending processes

Next, if using a laptop as a node, open the "/etc/systemd/logind.conf" file with a text editor, such as `vim`:

```
sudo vim /etc/systemd/logind.conf
```

Set the `HandleLidSwitch` option to ignore. It will likely be the only uncommented line in the file:

```
HandleLidSwitch=ignore
```

Save the file, and restart the node. Now, the laptop lid can be closed without the computer suspending.

### Configuring zsh

Next, install `zsh` and `zsh-syntax-highlighting`:

```
nala install zsh zsh-syntax-highlighting
```

Set the user's preferred shell to `zsh` with the following command:

```
chsh -s /bin/zsh
```

Now, clone the Powerlevel10k `git` repository into the "~/.local/src" directory:

```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.local/src/powerlevel10k
```

In order to quickly download and install the fonts necessary to make Powerlevel10k work, run the following script in the terminal:

```
fonts=(
"MesloLGS_NF_Regular.ttf"
"MesloLGS_NF_Bold.ttf"
"MesloLGS_NF_Italic.ttf"
"MesloLGS_NF_Bold_Italic.ttf"
)

echo "There are $(echo ${#fonts[@]}) fonts to install. It shouldn't take long."
echo "Elevated privileges are required to properly install the fonts. Please enter your password if asked."

sudo mkdir -p /usr/local/share/fonts/m

for font in "${fonts[@]}"; do
    webfont=$(echo $font | sed 's/_/%20/g')
    [ ! -e /usr/local/share/fonts/m/$font ] \
        && echo "now installing $font ..." \
        && sudo curl -L https://github.com/romkatv/powerlevel10k-media/raw/master/$webfont -o /usr/local/share/fonts/m/$font > /dev/null 2>&1
done
```

Since personal dotfiles should already exist in the `~/.local/src/dotfiles` directory, use `stow` to deploy them, or symlink the `~/.config/zsh/.zshrc` and `~/.config/zsh/.p10k.zsh` files into the correct directory. To use `stow`, first symlink the dotfiles into the home directory:

```
ln -s ~/.local/src/dotfiles ~/.dotfiles
```

Next, change directory into the new "~/.dotfiles" and deploy the dotfiles using `stow`. If there are any conflicts, be sure to address them:

```
cd ~/.dotfiles && stow .
```

If desired, change the color of the user's prompt by editing the `POWERLEVEL9K_USER_FOREGROUND` option in the "~/.config/zsh/.p10k.zsh" file.

Also, it is possible to change the prompt's OS icon to a Bitcoin logo. Add the following to the `os_icon: os identifier` section:

```
typeset -g POWERLEVEL9K_OS_ICON_CONTENT_EXPANSION=''
```

Note: the emoji in the above command may not render properly on GitHub. However, if the correct fonts are installed on the computer, then the configuration should work as intended.

Begin using the `zsh` shell with the following command.

```
zsh
```

### Finalizing initial configuration

Create a new directory "/data". This directory will contain symlinks to all configuration directories for all the services that will be installed on the node.

```
sudo mkdir /data && sudo chown $USERNAME: /data
```

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
systemctl enable ufw
```

To check and confirm the firewall rules, use the following command:

```
sudo ufw status
```

## Other security settings

Next, install `fail2ban`, a package designed to temporarily ban users who fail to login too many times in a short period of time. No configuration is needed besides installing the package.

```
nala install fail2ban
```

Open the "/etc/security/limits.d/90-limits.conf" file:

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

Next, open the "/etc/pam.d/common-session" file:

```
sudo vim /etc/pam.d/common-session
```

To the end of the file, append the following line:

```
session required    pam_limits.so
```

Open the very similarly named "/etc/pam.d/common-session-noninteractive" file:

```
sudo vim /etc/pam.d/common-session-noninteractive
```

To the end of the file, append the same line as with the other PAM file:

```
session required    pam_limits.so
```

## Configuring nginx

Now, install `nginx`:

```
nala install nginx
```

Configure an SSL certificate for the server:

```
sudo openssl req -x509 -nodes -newkey rsa:4096 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -subj "/CN=localhost" -days 365000
```

Open the "/etc/nginx/nginx.conf" file with a text editor:

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
nala install libnginx-mod-stream
```

## Configuring Tor

Now, install `tor`:

```
nala install tor
```

Open the "/etc/tor/torrc" file with a text editor:

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

First, open the "/etc/tor/torrc" file with a text editor:

```
sudo vim /etc/tor/torrc
```

In the section marked as "just for location-hidden services", add the following four lines:

```
# Hidden Service for $SERVICE_NAME
HiddenServiceDir /var/lib/tor/hidden_service_$SERVICENAME
HiddenServiceVersion 3
HiddenServicePort $PORT_OF_SERVICE 127.0.0.1:$PORT_OF_SERVICE
```

Exit the file, and reload the `tor` configurations with the following command:

```
systemctl reload tor
```

Use `cat` to output the file contents in the corresponding directory to obtain the onion link to the service:

```
sudo cat /var/lib/tor/hidden_service_$SERVICENAME/hostname
```

Now, it is possible to access that service remotely using Tor.

## Installing Bitcoin Core

The first step to installing Bitcoin Core is to download the installation files.

Change directory to the "/tmp" directory, so the files don't clutter the system:

```
cd /tmp
```

Next, grab the [download links](https://bitcoincore.org/en/download/) to the corresponding files, and download them using `curl -LJO`.

Note: `$VERSION` represents the current Bitcoin Core version (27.0, as of writing), and `$ARCHITECTURE` represents the CPU architecture (`x86_64` for laptops, and `aarch64` for Raspberry Pi). These can be set as variables, in the following way:

```
VERSION="27.0"
ARCHITECTURE="x86_64"
```

First, download the package archive:

```
curl -LJO https://bitcoincore.org/bin/bitcoin-core-$VERSION/bitcoin-$VERSION-$ARCHITECTURE-linux-gnu.tar.gz
```

Next, download the list of hashes for the binary archives:

```
curl -LJO https://bitcoincore.org/bin/bitcoin-core-$VERSION/SHA256SUMS
```

Finally, download the GPG signatures for those hashes:

```
curl -LJO https://bitcoincore.org/bin/bitcoin-core-$VERSION/SHA256SUMS.asc
```

Now, run a SHA256 checksum verification on the Bitcoin Core archive with the following:

```
sha256sum --ignore-missing --check SHA256SUMS
```

Next, grab the public keys (pubkeys) of the Bitcoin Core maintainers to verify the validity of the signatues:

```
curl -s "https://api.github.com/repositories/355107265/contents/builder-keys" | grep download_url | grep -oE "https://[a-zA-Z0-9./-]+" | while read url; do curl -s "$url" | gpg --import; done
```

Now that the pubkeys have been added to the GPG keyring, verify the "SHA256SUMS.asc" file:

```
gpg --verify SHA256SUMS.asc
```

With the signatures verified, it is now time to extract the compressed Bitcoin Core archive:

```
tar -xvf bitcoin-$VERSION-$ARCHITECTURE-linux-gnu.tar.gz
```

Install the extracted files into "/usr/local/bin" directory:

```
sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-$VERSION/bin/*
```

Confirm that the binary has been installed correctly by running the following command:

```
bitcoin-cli --version
```

### Configuring Bitcoin Core

Now that Bitcoin Core is installed, create a new user to manage the Bitcoin service:

```
sudo useradd -G debian-tor -s /bin/bash -m bitcoin
```

Note that the `-G` option directs the "bitcoin" user to be added to the "debian-tor" group upon creation, and the `-s` option changes the user's default shell to `/bin/bash`. The `-m` option directs the command to create a home directory for the new user.

Next, add the main admin user to the "bitcoin" group, so the user can manage the Bitcoin service as well:

```
sudo usermod -aG bitcoin $USERNAME
```

To confirm the change, check the groups of that user:

```
groups $USERNAME
```

Next, create a "/data/bitcoin" directory in the "/data" folder:

```
mkdir -pv /data/bitcoin
```

Change the ownership of that new directory so that the "bitcoin" user owns the directory and its contents:

```
sudo chown -R bitcoin: /data/bitcoin
```

Switch users to the "bitcoin" user:

```
sudo su - bitcoin
```

Now, create a symbolic link so that all Bitcoin Core configuration files are accessible from the "/data/bitcoin" directory:

```
ln -s /data/bitcoin ~/.bitcoin
```

Next, change directory into the "~/.bitcoin" directory and download a python file that will allow the user to generate a cookie file, for authentication purposes:

```
cd ~/.bitcoin && curl -LJO https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py
```

Now, create the new file "~/.bitcoin/bitcoin.conf" and add the following content to it:

```
# $HOSTNAME: bitcoind configuration
# /home/bitcoin/.bitcoin/bitcoin.conf

# Bitcoin daemon
server=1
txindex=1

# Allow creation of legacy wallets (required for JoinMarket)
deprecatedrpc=create_bdb

# Network
listen=1
listenonion=1
proxy=127.0.0.1:9050
bind=127.0.0.1

# Activate v2 P2P
v2transport=1

# Connections
rpcauth=<replace with authentication hash generated by rpcauth.py in the next step>
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28333
whitelist=download@127.0.0.1        # for Electrs

# Mempool
#maxmempool=1000

# Raspberry Pi optimizations
#maxconnections=40
#maxuploadtarget=5000

# Initial block download optimizations
dbcache=2000
blocksonly=1
```

Note that `$HOSTNAME` can be the hostname of the node, and `rpcauth` will be generated in the following step.

Now, generate the authentication hash for the `rpcauth` line of the "bitcoin.conf" file.

Since the "bitcoin" user is using `bash` as the default shell, use the following command with a space in front of it, so that the command is not saved into the command history. For reference, `$USERNAME` is the username of the main admin account being used, and `$PASSWORD` is a new password created for securing the Bitcoin Core RPC client.

```
 python3 rpcauth.py $USERNAME $PASSWORD
```

Copy the line generated by the python script and add it to the "bitcoin.conf" file, in place of the existing `rpcauth` line.

Next, change the permissions of the "bitcoin.conf" file, so only the "bitcoin" user and members of the "bitcoin" group can read it:

```
chmod 640 ~/.bitcoin/bitcoin.conf
```

With all of this completed, test the Bitcoin Core install by running the Bitcoin daemon:

```
bitcoind
```

After a minute or so, it should be clear whether or not `bitcoind` is running as intended. Stop the process for now.

Next, change permissions on the "debug.log" file:

```
chmod g+r /data/bitcoin/debug.log
```

Log out of the "bitcoin" user, and back to the main admin user. Symlink the "/data/bitcoin" directory into the main user's directory:

```
ln -s /data/bitcoin ~/.bitcoin
```

Now, create a `bitcoind` service file, so that the `bitcoind` process can run automatically when the node reboots:

```
sudo vim /etc/systemd/system/bitcoind.service
```

Add the following content to the new file:

```
# $HOSTNAME: systemd unit for bitcoind
# /etc/systemd/system/bitcoind.service

[Unit]
Description=Bitcoin daemon
After=network.target

[Service]

# Service execution
###################

ExecStart=/usr/local/bin/bitcoind -daemon \
                                  -pid=/run/bitcoind/bitcoind.pid \
                                  -conf=/home/bitcoin/.bitcoin/bitcoin.conf \
                                  -datadir=/home/bitcoin/.bitcoin \
                                  -startupnotify="chmod g+r /home/bitcoin/.bitcoin/.cookie"

# Process management
####################
Type=forking
PIDFile=/run/bitcoind/bitcoind.pid
Restart=on-failure
TimeoutSec=300
RestartSec=30

# Directory creation and permissions
####################################
User=bitcoin
UMask=0027

# /run/bitcoind
RuntimeDirectory=bitcoind
RuntimeDirectoryMode=0710

# Hardening measures
####################
# Provide a private /tmp and /var/tmp.
PrivateTmp=true

# Mount /usr, /boot/ and /etc read-only for the process.
ProtectSystem=full

# Disallow the process and all of its children to gain
# new privileges through execve().
NoNewPrivileges=true

# Use a new /dev namespace only populated with API pseudo devices
# such as /dev/null, /dev/zero and /dev/random.
PrivateDevices=true

# Deny the creation of writable and executable memory mappings.
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
```

Again, `$HOSTNAME` can be the hostname of the node.

Enable this new service with the following command:

```
systemctl enable bitcoind
```

To confirm that everything is working as intended, reboot the node:

```
reboot now
```

After logging back into the node, confirm that the `bitcoind` service is running with the following:

```
systemctl status bitcoind
```

In addition, confirm the permissions for the Bitcoin "cookie" file are "640" ("r+w" for the user, and "r" for the group):

```
ls ~/.bitcoin/.cookie
```

With `bitcoind` now running as intended, let the process run for a few days (now, up to about a week) in order to complete the initial sync.

### Checking on Bitcoin Core during the intial sync

While waiting, there are a few ways to get information about the Initial Blockchain Download (IBD). One way is to use the following command:

```
bitcoin-cli getblockchaininfo
```

This command gives a lot of output about the current status of the initial sync. It can be very useful, but is sometimes too verbose for a more seasoned node-runner.

Another way to check the status of the initial sync is with the following command:

```
tail -f ~/.bitcoin/debug.log
```

This command will use `tail` to check the last lines of the Bitcoin Core log file. The `-f` option represents "follow", and tells `tail` to continue to read the last lines until cancelled. Again, very useful, but sometimes too verbose.

The best way to query the process is with the following:

```
bitcoin-cli getblockchaininfo | grep blocks
```

This command will only output the line from `bitcoin-cli getblockchaininfo` that indicates the current block header. This is a quick way for someone who knows the current block height to get an idea of how far away their node is from syncing.

### Re-configuring Bitcoin Core after the initial sync

Once Bitcoin Core has completed its initial sync, be sure to edit the "bitcoin.conf" file to make the following adjustments.

First, uncomment the following line:

```
# Mempool
#maxmempool=1000
```

Next, comment out the following two lines:

```
# Initial block download optimizations
dbcache=2000
blocksonly=1
```

To push these changes, restart the `bitcoind` service:

```
systemctl restart bitcoind
```

Congrats on syncing Bitcoin Core! The next step is to install a transaction indexer to make use of other Bitcoin services, such as wallet software and timechain explorers.

## Installing Fulcrum (Bitcoin transaction indexer)

To install Fulcrum, a Bitcoin transaction indexer, perform the following steps.

First, add an entry to `ufw` to allow SSL connections to Fulcrum:

```
sudo ufw allow 50002/tcp comment "allow Fulcrum SSL"
```

Also, add an entry to `ufw` to allow TCP connections to Fulcrum:

```
sudo ufw allow 50001/tcp comment "allow Fulcrum TCP"
```

Next, edit the "bitcoin.conf" file:

```
sudo vim ~/.bitcoin/bitcoin.conf
```

To the "Connections" section, add the following:

```
zmqpubhashblock=tcp://127.0.0.1:8433
```

With that configuration added, restart the `bitcoind` service:

```
systemctl restart bitcoind
```

Now, it's time to install Fulcrum. Change directory to the "/tmp" directory, so the files don't clutter the system:

```
cd /tmp
```

Next, grab the [download links](https://github.com/cculianu/Fulcrum/releases) to the corresponding files, and download them using `curl -LJO`.

Note: `$VERSION` represents the current Fulcrum version (1.10.0, as of writing), and `$ARCHITECTURE` represents the CPU architecture (`x86_64` for laptops, and `arm64` for Raspberry Pi). This can be set as variables, in the following way:

```
VERSION="1.10.0"
ARCHITECTURE="x86_64"
```

First, download the package archive:

```
curl -LJO https://github.com/cculianu/Fulcrum/releases/download/v$VERSION/Fulcrum-$VERSION-$ARCHITECTURE-linux.tar.gz
```

Next, download the list of hashes for the binary archives:

```
curl -LJO https://github.com/cculianu/Fulcrum/releases/download/v$VERSION/Fulcrum-$VERSION-shasums.txt
```

Finally, download the GPG signatures for those hashes:

```
curl -LJO https://github.com/cculianu/Fulcrum/releases/download/v$VERSION/Fulcrum-$VERSION-shasums.txt.asc
```

Now, run a SHA256 checksum verification on the Fulcrum archive with the following:

```
sha256sum --check --ignore-missing Fulcrum-$VERSION-shasums.txt
```

Next, grab the public keys (pubkeys) of the Fulcrum developer to verify the validity of the signatues:

```
curl https://raw.githubusercontent.com/Electron-Cash/keys-n-hashes/master/pubkeys/calinkey.txt | gpg --import
```

Now that the pubkeys have been added to the GPG keyring, verify the "Fulcrum-$VERSION-shasums.txt.asc" file:

```
gpg --verify Fulcrum-$VERSION-shasums.txt.asc
```

With the signatures verified, it is now time to extract the compressed Fulcrum archive:

```
tar -xvf Fulcrum-$VERSION-$ARCHITECTURE-linux.tar.gz
```

Install the extracted files into "/usr/local/bin" directory:

```
sudo install -m 0755 -o root -g root -t /usr/local/bin Fulcrum-1.10.0-x86_64-linux/Fulcrum Fulcrum-1.10.0-x86_64-linux/FulcrumAdmin
```

Confirm that the binary has been installed correctly by running the following command:

```
Fulcrum --version
```

### Configuring zram-swap

First, install the `libssl-dev` package:

```
nala update && nala install libssl-dev
```

Next, clone the git repo for `zram-swap`:

```
cd ~/.local/src && git clone https://github.com/foundObjects/zram-swap
```

Enter the `zram-swap` directory and install the package:

```
cd zram-swap && sudo ./install.sh
```

With `zram-swap` installed, edit "/etc/default/zram-swap":

```
sudo vim /etc/default/zram-swap
```

Comment out the first line, and add the second:

```
#_zram_fraction="1/2"
_zram_fixedsize="10G"
```

Note: the fixed size of the ZRAM should be congruent with the RAM on the computer. For a computer with 8GB RAM, it is acceptable to enter 10GB. The number set for `_zram_fixedsize` should be anywhere from 100-125% of the computer's RAM.

To confirm that `zram-swap` is working as intended, first use `cat` on the following path:

```
sudo cat /proc/swaps
```

This should show memory usage *before* `zram-swap` is enabled. Take note of the output.

Next, edit the "/etc/sysctl.conf" file:

```
sudo vim /etc/sysctl.conf
```

Add the following lines to the config file:

```
vm.vfs_cache_pressure=500
vm.swappiness=100
vm.dirty_background_ratio=1
vm.dirty_ratio=50
```

After adding those lines to the config file, reload `sysctl` with the following command:

```
sudo sysctl --system
```

Now, restart the `zram-swap` service:

```
systemctl restart zram-swap
```

Use `cat` on "/proc/swaps" again; `/dev/zram0` should now be listed, with a higher priority than other entries:

```
sudo cat /proc/swaps
```

As a final confirmation, run `systemctl status` on the `zram-swap` service:

```
systemctl status zram-swap
```

### Configuring Fulcrum

Now that Fulcrum is installed, create a new user to manage the Fulcrum service:

```
sudo useradd -G bitcoin -s /bin/bash -m fulcrum
```

Note that the `-G` option directs the "fulcrum" user to be added to the "bitcoin" group upon creation, and the `-s` option changes the user's default shell to `/bin/bash`. The `-m` option directs the command to create a home directory for the new user.

Next, create a "/data/fulcrum" and "/data/fulcrum/fulcrum_db" directory in the "/data" folder:

```
mkdir -pv /data/fulcrum/fulcrum_db
```

Change the ownership of that new directory so that the "fulcrum" user owns the directory and its contents:

```
sudo chown -R fulcrum: /data/fulcrum
```

Switch users to the "fulcrum" user:

```
su - fulcrum
```

Now, create a symbolic link so that all Fulcrum configuration files are accessible from the "/data/fulcrum" directory:

```
sudo ln -s /data/fulcrum ~/.fulcrum
```

Change directory into the new "~/.fulcrum" directory, and create a new SSL key for the Fulcrum service:

```
cd ~/.fulcrum && openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout fulcrum.key -out fulcrum.crt
```

Next, create and open the "/data/fulcrum/fulcrum.conf" file with a text editor:

```
# $HOSTNAME: fulcrum configuration
# /data/fulcrum/fulcrum.conf

# Bitcoin Core settings
bitcoind = 127.0.0.1:8332
rpccookie = /home/bitcoin/.bitcoin/.cookie

# Fulcrum server settings
datadir = /data/fulcrum/fulcrum_db
cert = /data/fulcrum/fulcrum.crt
key = /data/fulcrum/fulcrum.key
ssl = 0.0.0.0:50002
tcp = 0.0.0.0:50001
peering = false

# Optimizations
bitcoind_timeout = 600
bitcoind_clients = 1
worker_threads = 1
db_mem = 1024.0

# 8GB RAM (default)
db_max_open_files = 400
fast-sync = 2048

# 16GB RAM (comment the last two lines and uncomment the next)
#db_max_open_files = 800
#fast-sync = 4096
```

Note: `$HOSTNAME` can be the hostname of the node. Also, note that there are configurations at the bottom that depend on the amount of RAM the system has. Use the settings that make the most sense for the computer acting as the node.

Log out of the "fulcrum" user, and back to the main admin user.

Now, create a Fulcrum service file, so that the Fulcrum process can run automatically when the node reboots:

```
sudo vim /etc/systemd/system/fulcrum.service
```

Add the following content to the new file:

```
# $HOSTNAME: systemd unit for Fulcrum
# /etc/systemd/system/fulcrum.service

[Unit]
Description=Fulcrum
PartOf=bitcoind.service
After=bitcoind.service
StartLimitBurst=2
StartLimitIntervalSec=20

[Service]
ExecStart=/usr/local/bin/Fulcrum /data/fulcrum/fulcrum.conf
KillSignal=SIGINT
User=fulcrum
Type=exec
TimeoutStopSec=300
RestartSec=30
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Again, `$HOSTNAME` can be the hostname of the node.

Enable this new service with the following command:

```
systemctl enable fulcrum
```

Now, start the service with the following command:

```
systemctl start fulcrum
```

Check its status with the `systemctl status` command:

```
systemctl status fulcrum
```

To see the output of the Fulcrum service, run the following:

```
sudo journalctl -fu fulcrum
```

**Note: this process will likely take days to run, as Fulcrum is now indexing all transactions throughout the *entire* Bitcoin timechain. It is strongly suggested that the node stay online through this entire process, and that Fulcrum is not stopped or rebooted until the process completes.**

In the case that Fulcrum *is* stopped or rebooted, or the node itself is rebooted, data may become corrupted. If this happens, delete the "/data/fulcrum/fulcrum_db" directory and run `mkdir /data/fulcrum/fulcrum_db` to restore an empty directory, and begin the process again.

### Re-configuring Fulcrum after the initial sync

Once Fulcrum completes its initial syncing and compacting, `zram-swap` can be reconfigured. Open the "/etc/default/zram-swap" file in a text editor:

```
sudo vim /etc/default/zram-swap
```

Perform the following adjustments:

```
_zram_fraction="1/2"
#_zram_fixedsize="10G"
```

After adding those lines to the config file, reload `sysctl` with the following command:

```
sudo sysctl --system
```

Now, restart the `zram-swap` service:

```
systemctl restart zram-swap
```

To confirm that `zram-swap` has been modified correctly, use `cat` on the following path:

```
sudo cat /proc/swaps
```

### Adding a hidden service for Fulcrum

As is described in the section on [adding a hidden service](#adding-a-hidden-service), follow these steps to create a hidden service for Fulcrum:

First, open the "/etc/tor/torrc" file in a text editor:

```
sudo vim /etc/tor/torrc
```

In the section marked as "just for location-hidden services", add the following lines:

```
# Hidden Service for Fulcrum SSL
HiddenServiceDir /var/lib/tor/hidden_service_fulcrum_ssl
HiddenServiceVersion 3
HiddenServicePort 50002 127.0.0.1:50002

# Hidden Service for Fulcrum TCP
HiddenServiceDir /var/lib/tor/hidden_service_fulcrum_tcp
HiddenServiceVersion 3
HiddenServicePort 50001 127.0.0.1:50001
```

Exit the file, and reload the `tor` configurations with the following command:

```
systemctl reload tor
```

Use `cat` to output the file contents in the corresponding directory to obtain the onion link to the service:

```
sudo cat /var/lib/tor/hidden_service_fulcrum_ssl/hostname
```

Or:

```
sudo cat /var/lib/tor/hidden_service_fulcrum_tcp/hostname
```

Now, it is possible to access Fulcrum remotely using Tor.

### Editing the banner for Fulcrum

When using a wallet software such as [Sparrow Wallet](https://github.com/sparrowwallet/sparrow), a banner will display when connecting the wallet to the indexer. Using a webpage such as [this one](https://patorjk.com/software/taag/#p=display&f=Slant&t=Fulcrum), it is possible to create custom ASCII art to be displayed when the connection is tested.

To do this, create a "/data/fulcrum/banner.txt" file:

```
sudo vim /data/fulcrum/banner.txt
```

Add the custom ASCII art to this file, then save and exit. To make sure the banner displays properly, change the owner of the file to the "fulcrum" user.

```
sudo chown -R fulcrum: /data/fulcrum/banner.txt
```

Next, open the "/data/fulcrum/fulcrum.conf" file.

```
sudo vim /data/fulcrum/fulcrum.conf
```

Add the following configuration to the end of the file:

```
# Banner path
banner = /data/fulcrum/banner.txt
```

Now, restart the Fulcrum service:

```
systemctl restart fulcrum
```

When connecting to Fulcrum using wallet software, the banner should display!

## Installing Mempool.space (Timechain explorer)

[Mempool.space](https://mempool.space) is a Bitcoin timechain explorer and visualizer. While the links directs to a publicly hosted version of the site, it is possible to set up a self-hosted version on the Bitcoin node that references the node, preventing any forms of data leak.

To install Mempool, perform the following steps.

First, install `nodejs`.

Note: the best way to go about this on Debian is **not** to use the `nodejs` package found in the Debian package repositories. Instead, use the following curl command to download and prepare a version of `nodejs` provided by the Node.js maintainers.

```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

Now, install `nodejs`:

```
nala install nodejs
```

To confirm that `nodejs` installed correctly, check the node's `node` version:

```
node -v
```

It is also a good idea to check the node's `npm` version at this time:

```
npm --version
```

Now, add a `ufw` firewall rule to allow connections to Mempool:

```
sudo ufw allow 4081/tcp comment 'allow Mempool SSL'
```

Confirm the rule was entered correctly with the following:

```
sudo ufw status
```

Next, create a new user to manage the Mempool service:

```
sudo useradd -G bitcoin -s /bin/bash -m mempool
```

Note that the `-G` option directs the "mempool" user to be added to the "bitcoin" group upon creation, and the `-s` option changes the user's default shell to `/bin/bash`. The `-m` option directs the command to create a home directory for the new user.

Switch users to the "mempool" user:

```
su - mempool
```

Clone the Mempool project into the "mempool" user's home directory, then change directory into it:

```
git clone https://github.com/mempool/mempool && cd mempool
```

Using the [Mempool's GitHub page](https://github.com/mempool/mempool) as a reference, direct the `git checkout` command to checkout the latest release version (as of writing, v2.5.0):

```
git checkout v2.5.0
```

Log out of the "mempool" user, and back to the main admin user.

### Configuring Mempool

As the main admin user, install the MariaDB database package:

```
nala update && nala install mariadb-server mariadb-client
```

Use the following command to generate a randomized 25 character password; this password will be used to secure the SQL database:

```
tr -dc A-Za-z0-9 </dev/urandom | head -c 25; echo
```

Now, enter the shell for the SQL database:

```
sudo mysql
```

Run the following commands:

```
CREATE DATABASE mempool;
GRANT ALL PRIVILEGES ON mempool.* TO 'mempool'@'127.0.0.1' IDENTIFIED BY "PASSWORD_GENERATED_MOMENTS_AGO";
FLUSH PRIVILEGES;
exit;
```

Switch users to the "mempool" user:

```
su - mempool
```

#### Configuring the Mempool backend

Change directory into "~/mempool/backend", then install and build the backend:

```
cd ~/mempool/backend && npm install --prod && npm run build
```

Create the configuration file in this directory by opening "mempool-config.json" in a text editor:

```
vim mempool-config.json
```

Add the following content to the file:

```
{
    "MEMPOOL": {
        "NETWORK": "mainnet",
            "BACKEND": "electrum",
            "HTTP_PORT": 8999,
            "SPAWN_CLUSTER_PROCS": 0,
            "API_URL_PREFIX": "/api/v1/",
            "POLL_RATE_MS": 2000,
            "CACHE_DIR": "./cache",
            "CLEAR_PROTECTION_MINUTES": 20,
            "RECOMMENDED_FEE_PERCENTILE": 50,
            "BLOCK_WEIGHT_UNITS": 4000000,
            "INITIAL_BLOCKS_AMOUNT": 8,
            "MEMPOOL_BLOCKS_AMOUNT": 8,
            "PRICE_FEED_UPDATE_INTERVAL": 3600,
            "USE_SECOND_NODE_FOR_MINFEE": false,
            "EXTERNAL_ASSETS": []
    },

    "CORE_RPC": {
        "HOST": "127.0.0.1",
        "PORT": 8332,
        "USERNAME": "$MAIN_ADMIN_USER",
        "PASSWORD": "$BITCOIN_RPC_PASSWORD"
    },

    "ELECTRUM": {
        "HOST": "127.0.0.1",
        "PORT": 50002,
        "TLS_ENABLED": true
    },

    "DATABASE": {
        "ENABLED": true,
        "HOST": "127.0.0.1",
        "PORT": 3306,
        "USERNAME": "mempool",
        "PASSWORD": "$MEMPOOL_SQL_DATABASE_PASSWORD",
        "DATABASE": "mempool"
    },

    "SOCKS5PROXY": {
        "ENABLED": true,
        "USE_ONION": true,
        "HOST": "127.0.0.1",
        "PORT": 9050
    },

    "PRICE_DATA_SERVER": {
        "TOR_URL": "http://wizpriceje6q5tdrxkyiazsgu7irquiqjy2dptezqhrtu7l2qelqktid.onion/getAllMarketPrices"
    }
}
```

Note: remember to change `$MAIN_ADMIN_USER` and `$BITCOIN_RPC_PASSWORD` to the values set when configuring Bitcoin Core. Also, change `$MEMPOOL_SQL_DATABASE_PASSWORD` to the password set earlier, when creating the SQL database.

With the file configured properly, start the backend and confirm that it's running as intended:

```
npm run start
```

After ~30 seconds, output like the following should appear:

```
Updating mempool
Mempool updated in 0.xyz seconds
```

Exit the `npm` process.

#### Configuring the Mempool frontend

While still logged in as the "mempool" user, change directory into "~/mempool/frontend", then install and build the frontend:

```
cd ~/mempool/frontend && npm install --prod && npm run build
```

Log out of the "mempool" user, and back to the main admin user.

Move the frontend output into the webroot directory, then change the owner to the "www-data" user:

```
sudo rsync -av --delete /home/mempool/mempool/frontend/dist/mempool/ /var/www/mempool/ && sudo chown -R www-data: /var/www/mempool
```

#### Finalizing the Mempool configuration

Earlier, the file "mempool-config.json" was created, and it contains sensitive credentials to the Bitcoin Core RPC process. To limit who can see and interact with those credentials, change the permissions on the file so that only its owner (the "mempool" user) can read or write that file:

```
sudo chmod 600 /home/mempool/mempool/backend/mempool-config.json
```

Next, open the "/etc/nginx/sites-available/mempool-ssl.conf" file in a text editor:

```
sudo vim /etc/nginx/sites-available/mempool-ssl.conf
```

Add the following content to the file:

```
proxy_read_timeout 300;
proxy_connect_timeout 300;
proxy_send_timeout 300;

map $http_accept_language $header_lang {
    default en-US;
    ~*^en-US en-US;
    ~*^en en-US;
    ~*^ar ar;
    ~*^ca ca;
    ~*^cs cs;
    ~*^de de;
    ~*^es es;
    ~*^fa fa;
    ~*^fr fr;
    ~*^ko ko;
    ~*^it it;
    ~*^he he;
    ~*^ka ka;
    ~*^hu hu;
    ~*^mk mk;
    ~*^nl nl;
    ~*^ja ja;
    ~*^nb nb;
    ~*^pl pl;
    ~*^pt pt;
    ~*^ro ro;
    ~*^ru ru;
    ~*^sl sl;
    ~*^fi fi;
    ~*^sv sv;
    ~*^th th;
    ~*^tr tr;
    ~*^uk uk;
    ~*^vi vi;
    ~*^zh zh;
    ~*^hi hi;
}

map $cookie_lang $lang {
    default $header_lang;
    ~*^en-US en-US;
    ~*^en en-US;
    ~*^ar ar;
    ~*^ca ca;
    ~*^cs cs;
    ~*^de de;
    ~*^es es;
    ~*^fa fa;
    ~*^fr fr;
    ~*^ko ko;
    ~*^it it;
    ~*^he he;
    ~*^ka ka;
    ~*^hu hu;
    ~*^mk mk;
    ~*^nl nl;
    ~*^ja ja;
    ~*^nb nb;
    ~*^pl pl;
    ~*^pt pt;
    ~*^ro ro;
    ~*^ru ru;
    ~*^sl sl;
    ~*^fi fi;
    ~*^sv sv;
    ~*^th th;
    ~*^tr tr;
    ~*^uk uk;
    ~*^vi vi;
    ~*^zh zh;
    ~*^hi hi;
}

server {
    listen 4081 ssl;
    listen [::]:4081 ssl;
    server_name _;
    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_session_timeout 4h;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers on;

    include /etc/nginx/snippets/nginx-mempool.conf;
}
```

Next, create a symlink from the "sites-available" directory to the "sites-enabled" directory:

```
sudo ln -sf /etc/nginx/sites-available/mempool-ssl.conf /etc/nginx/sites-enabled/
```

Now, copy the Mempool nginx configuration file into the "/etc/nginx/snippets" directory:

```
sudo rsync -av /home/mempool/mempool/nginx-mempool.conf /etc/nginx/snippets
```

Back up the current `nginx` configuration file by moving it to a different path:

```
sudo mv -v /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

Open the "/etc/nginx/nginx.conf" in a text editor to create a new file:

```
sudo vim /etc/nginx/nginx.conf
```

Add the following contents to the file:

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers on;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    gzip on;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

stream {
    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 4h;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers on;
    include /etc/nginx/streams-enabled/*.conf;
}
```

Test the `nginx` configuration file with the following command:

```
sudo nginx -t
```

Now, use `systemctl reload` to update the `nginx` configuration:

```
systemctl reload nginx
```

Now, create a Mempool service file, so that the Mempool process can run automatically when the node reboots:


```
sudo vim /etc/systemd/system/mempool.service
```

Add the following content to the file:

```
# $HOSTNAME: systemd unit for Mempool
# /etc/systemd/system/mempool.service

[Unit]
Description=mempool
After=bitcoind.service

[Service]
WorkingDirectory=/home/mempool/mempool/backend
ExecStart=/usr/bin/node --max-old-space-size=2048 dist/index.js
User=mempool

# Restart on failure but no more than default times (DefaultStartLimitBurst=5) every 10 minutes (600 seconds). Otherwise stop
Restart=on-failure
RestartSec=600

# Hardening measures
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true

[Install]
WantedBy=multi-user.target
```

Note: `$HOSTNAME` can be the hostname of the node.

Enable the new Mempool service to start when the system boots, and start the service:

```
systemctl enable mempool && systemctl start mempool
```

Check the Mempool service logs to see it working:

```
sudo journalctl -fu mempool
```

Now, the site should be available at `https://HOSTNAME.TLD:4081`. In this case, `HOSTNAME` can also be the IP address of the node.

### Adding a hidden service for Mempool

As is described in the section on [adding a hidden service](#adding-a-hidden-service), follow these steps to create a hidden service for Mempool:

First, open the "/etc/tor/torrc" file in a text editor:

```
sudo vim /etc/tor/torrc
```

In the section marked as "just for location-hidden services", add the following lines:

```
# Hidden Service for Mempool
HiddenServiceDir /var/lib/tor/hidden_service_mempool
HiddenServiceVersion 3
HiddenServicePort 443 127.0.0.1:4081
```

Exit the file, and reload the `tor` configurations with the following command:

```
systemctl reload tor
```

Use `cat` to output the file contents in the corresponding directory to obtain the onion link to the service:

```
sudo cat /var/lib/tor/hidden_service_mempool/hostname
```

Now, it is possible to access Mempool remotely using Tor.

## References

- [Raspibolt documentation](https://raspibolt.org/)
    - The primary reference for many aspects of this guide.
- [YouTube - Ministry of Nodes - Ubuntu Node Box playlist](https://www.youtube.com/playlist?list=PLCRbH-IWlcW290O0N0lQV6efxuCA5Ja8c)
    - The original reference for some aspects of this guide.
