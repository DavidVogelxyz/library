Installing Fulcrum (Bitcoin transaction indexer)
================================================

[Back to the home page](README.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Installing Fulcrum](#installing-fulcrum)
    - [Configuring zram-swap](#configuring-zram-swap)
    - [Configuring Fulcrum](#configuring-fulcrum)
    - [Re-configuring Fulcrum after the initial sync](#re-configuring-fulcrum-after-the-initial-sync)
    - [Adding a hidden service for Fulcrum](#adding-a-hidden-service-for-fulcrum)
    - [Editing the banner for Fulcrum](#editing-the-banner-for-fulcrum)
- [References](#references)

Introduction
------------

Fulcrum is a Bitcoin transaction indexer. A transaction indexer is required for most Bitcoin wallet software, as well as the [Mempool timechain explorer](installing-mempool.md). This section of the guide will describe the process for configuring and installing the Fulcrum indexer.

Installing Fulcrum
------------------

To install Fulcrum, a Bitcoin transaction indexer, perform the following steps.

First, add an entry to `ufw` to allow TCP connections to Fulcrum:

```
sudo ufw allow 50001/tcp comment "allow Fulcrum TCP"
```

Also, add an entry to `ufw` to allow SSL connections to Fulcrum:

```
sudo ufw allow 50002/tcp comment "allow Fulcrum SSL"
```

Next, edit the `bitcoin.conf` file:

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

Now, it's time to install Fulcrum. Change directory to the `/tmp` directory, so the files don't clutter the system:

```
cd /tmp
```

Next, grab the [download links](https://github.com/cculianu/Fulcrum/releases) to the corresponding files, and download them using `curl -LJO`.

Note: `$VERSION` represents the current Fulcrum version (1.11.1, as of writing), and `$ARCHITECTURE` represents the CPU architecture (`x86_64` for laptops, and `arm64` for Raspberry Pi). This can be set as variables, in the following way:

```
VERSION="1.11.1"
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

Now that the pubkeys have been added to the GPG keyring, verify the `Fulcrum-$VERSION-shasums.txt.asc` file:

```
gpg --verify Fulcrum-$VERSION-shasums.txt.asc
```

With the signatures verified, it is now time to extract the compressed Fulcrum archive:

```
tar -xvf Fulcrum-$VERSION-$ARCHITECTURE-linux.tar.gz
```

Install the extracted files into `/usr/local/bin` directory:

```
sudo install -m 0755 -o root -g root -t /usr/local/bin Fulcrum-$VERSION-$ARCHITECTURE-linux/Fulcrum Fulcrum-$VERSION-$ARCHITECTURE-linux/FulcrumAdmin
```

Confirm that the binary has been installed correctly by running the following command:

```
Fulcrum --version
```

Configuring zram-swap
---------------------

First, install the `libssl-dev` package:

```
nala update && nala install -y libssl-dev
```

Next, clone the git repo for `zram-swap`:

```
cd ~/.local/src && git clone https://github.com/foundObjects/zram-swap
```

Enter the `zram-swap` directory and install the package:

```
cd zram-swap && sudo ./install.sh
```

With `zram-swap` installed, edit `/etc/default/zram-swap`:

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

Next, edit the `/etc/sysctl.conf` file:

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

Configuring Fulcrum
-------------------

Now that Fulcrum is installed, create a new user to manage the Fulcrum service:

```
sudo useradd -G bitcoin -s /bin/bash -m fulcrum
```

Note that the `-G` option directs the `fulcrum` user to be added to the `bitcoin` group upon creation, and the `-s` option changes the user's default shell to `/bin/bash`. The `-m` option directs the command to create a home directory for the new user.

Next, create a `/data/fulcrum` and `/data/fulcrum/fulcrum_db` directory in `/data`:

```
mkdir -pv /data/fulcrum/fulcrum_db
```

Change the ownership of that new directory so that the `fulcrum` user owns the directory and its contents:

```
sudo chown -R fulcrum: /data/fulcrum
```

Switch users to the `fulcrum` user:

```
su - fulcrum
```

Now, create a symbolic link so that all Fulcrum configuration files are accessible from the `/data/fulcrum` directory:

```
ln -s /data/fulcrum ~/.fulcrum
```

Change directory into the new `~/.fulcrum` directory, and create a new SSL key for the Fulcrum service:

```
cd ~/.fulcrum && openssl req -newkey rsa:4096 -new -nodes -x509 -days 36500 -keyout fulcrum.key -out fulcrum.crt
```

Next, create and open the `/data/fulcrum/fulcrum.conf` file with a text editor:

```
# <HOSTNAME>: fulcrum configuration
# /data/fulcrum/fulcrum.conf

# Bitcoin Core settings
bitcoind = 127.0.0.1:8332
rpccookie = /home/bitcoin/.bitcoin/.cookie

# Fulcrum server settings
datadir = /data/fulcrum/fulcrum_db
cert = /data/fulcrum/fulcrum.crt
key = /data/fulcrum/fulcrum.key
tcp = 0.0.0.0:50001
ssl = 0.0.0.0:50002
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

Note: `<HOSTNAME>` should be the hostname of the node. Also, note that there are configurations at the bottom that depend on the amount of RAM the system has. Use the settings that make the most sense for the computer acting as the node.

Log out of the `fulcrum` user, and back to the main admin user.

Now, create a Fulcrum service file, so that the Fulcrum process can run automatically when the node reboots:

```
sudo vim /etc/systemd/system/fulcrum.service
```

Add the following content to the new file:

```
# <HOSTNAME>: systemd unit for Fulcrum
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

Again, `<HOSTNAME>` should be the hostname of the node.

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

Re-configuring Fulcrum after the initial sync
---------------------------------------------

Once Fulcrum completes its initial syncing and compacting, `zram-swap` can be reconfigured. Open the `/etc/default/zram-swap` file in a text editor:

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

Adding a hidden service for Fulcrum
-----------------------------------

As is described in the section on [adding a hidden service](#adding-a-hidden-service), follow these steps to create a hidden service for Fulcrum:

First, open the `/etc/tor/torrc` file in a text editor:

```
sudo vim /etc/tor/torrc
```

In the section marked as "just for location-hidden services", add the following lines:

```
# Hidden Service for Fulcrum TCP
HiddenServiceDir /var/lib/tor/hidden_service_fulcrum_tcp
HiddenServiceVersion 3
HiddenServicePort 50001 127.0.0.1:50001

# Hidden Service for Fulcrum SSL
HiddenServiceDir /var/lib/tor/hidden_service_fulcrum_ssl
HiddenServiceVersion 3
HiddenServicePort 50002 127.0.0.1:50002
```

Exit the file, and reload the `tor` configurations with the following command:

```
systemctl reload tor
```

Use `cat` to output the file contents in the corresponding directory to obtain the onion link to the service:

```
sudo cat /var/lib/tor/hidden_service_fulcrum_tcp/hostname
```

Or:

```
sudo cat /var/lib/tor/hidden_service_fulcrum_ssl/hostname
```

Now, it is possible to access Fulcrum remotely using Tor.

Editing the banner for Fulcrum
------------------------------

When using a wallet software such as [Sparrow Wallet](https://github.com/sparrowwallet/sparrow), a banner will display when connecting the wallet to the indexer. Using a webpage such as [this one](https://patorjk.com/software/taag/#p=display&f=Slant&t=Fulcrum), it is possible to create custom ASCII art to be displayed when the connection is tested.

To do this, create a `/data/fulcrum/banner.txt` file:

```
sudo vim /data/fulcrum/banner.txt
```

Add the custom ASCII art to this file, then save and exit.

Note: it's also possible to display the version numbers for Bitcoin Core and Fulcrum in the banner. To do this, simply add these lines to the `banner.txt` file:

```
bitcoind version: $DAEMON_VERSION
fulcrum version: $SERVER_VERSION
```

To make sure the banner displays properly, change the owner of the file to the `fulcrum` user.

```
sudo chown -R fulcrum: /data/fulcrum/banner.txt
```

Next, open the `/data/fulcrum/fulcrum.conf` file.

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

References
----------

- [Raspibolt - Fulcrum server](https://raspibolt.org/guide/bonus/bitcoin/fulcrum.html)
    - Reference for initial setup of the Fulcrum indexer.
