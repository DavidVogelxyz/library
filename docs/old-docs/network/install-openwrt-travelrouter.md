Travel Router using a Raspberry Pi and OpenWrt
==============================================

#### **NB: For me, this guide is now broken. Due to a update to the `mt7601u-firmware` driver package on 2023 May 15, the USB wireless antenna I have for my travel routers no longer work. However, the previous version of the package (from 2022) works fine. Therefore, I recreate my travel routers from a backup image I made. This guide probably still works for other driver/antenna pairings.**

Menu
----

- [Start up](#start-up)
- [Edit config files, part 1/2](#edit-config-files-part-1)
  - [Network file](#network-file)
  - [Firewall file](#firewall-file)
- [Edit config files, part 2/2](#edit-config-files-part-2)
  - [Wireless file, part 1/2](#wireless-file-part-1)
  - [Configure the External Wireless Adapter](#configure-the-external-wireless-adapter)
  - [Package management](#package-management)
    - [Drivers](#drivers)
  - [Wireless file, part 2/2](#wireless-file-part-2)
- [VPN connection](#vpn-connection)
  - [Firewall](#firewall)
  - [Kill-switch](#kill-switch)
  - [Disable VPN kill-switch](#disable-vpn-kill-switch)
- [Using the Travel Router](#using-the-travel-router)
  - [Troubleshooting](#troubleshooting)
- [References](#references)

Start up
--------

After installing OpenWrt onto a SD card, and turning on the Raspberry Pi, connect using Ethernet and make the client's static IP `192.168.1.20` so it can talk to gateway `192.168.1.1`.

```
ssh root@192.168.1.1
```

Change password for root account.

```
passwd
```

Edit config files, part 1
-------------------------

```
cd /etc/config
```

Backup the following files:

```
cp -v firewall firewall-bkp

cp -v network network-bkp

cp -v wireless wireless-bkp
```

**NB: Normally, I use 'cp -v', but it doesn't seem to matter here.**

### Network file

Edit `/etc/config/network`.

```
vi network
```

Adjust `interface 'lan'` in the following ways:

1. Change `ipaddr` to something cool. Example: `10.11.12.1`
1. Change `netmask` if you know networking and need more than ~250 client addresses.
1. To the bottom, add `option force_link '1'`

Add a new interface and the following options with:

```
config interface 'wwan'
  option proto 'dhcp'
  option peendns '0'
  option dns '9.9.9.9 1.1.1.1'
```

Add another interface and the following options with:

```
config interface 'vpnclient'
  option ifname 'tun0'
  option proto 'none'
```

Save & exit.

### Firewall file

Edit `/etc/config/firewall`.

```
vi firewall
```

Adjust `config > zone > option > name > wan` by:
- Changing `option > input` to `ACCEPT`

Save & exit.

Reboot the router with:

```
reboot now
```

Edit config files, part 2
-------------------------

Change Ethernet adapter settings back to DHCP so it can be assigned an IP address by the gateway under the new IP scheme.

```
ssh root@10.11.12.1
```

### Wireless file, part 1

I made a decision to have the connection `public wireless <--> travel router` be mediated by the external wireless adapter, and the `travel router private LAN` to be mediated by the internal wireless adapter.

So far, I think this was a good choice. This section is where I make those edits.

Edit `/etc/config/wireless`.

```
vi /etc/config/wireless
```

Enable the wireless interface by **deleting** the line `radio0 > option disabled '0'`.

```
  option disabled '0'
```

Change the `encryption` to `psk2` and add a line with `option key` and supply a `$password`.

```
  option encryption 'psk2'
  option key '$password'
```

**NB: It seems that attempting to use a password that's too simple will result in OpenWrt not bringing up the wireless interface. If `wlan0` does not come up after the `wifi` command, try a stronger password.**

Now, commit the changes and enable wireless.

```
uci commit wireless

wifi
```

Now, you should be able to connect to the network you created by interfacing with the Raspberry Pi's internal wireless adapter (wlan0).

### Configure the External Wireless Adapter

Head over to the GUI.

```
Network > Wireless
```

Now, scan with `radio0`.

Connect to the correct wireless network.

Save & save & 'save & apply'.

### Package management

Now, head back to the command line via SSH.

```
opkg update
```

**NB: If `opkg update` is run before the OpenWrt device has a working WAN connection, then the update will always fail, even after connection to the internet is established. To fix, reboot the OpenWrt system.**

#### Drivers

After the package list is up-to-date, install the following packages.

```
opkg install kmod-usb-core kmod-usb-uhci kmod-usb-ohci kmod-usb2 usbutils openvpn-openssl luci-app-openvpn vim
```

When learning how to build travel routers, the following were the set of drivers to be installed, in order to cover most adapters.

```
opkg install kmod-rt2800-lib kmod-rt2800-usb kmod-rt2x00-lib kmod-rt2x00-usb
```

However, these were the only driver packages that I needed for my specific wireless adapter.

```
opkg install kmod-mt7601u mt7601u-firmware
```

**NB: I attempted to follow this guide through in 2023 June and hit a wall at this step. The `mt7601u-firmware` package was updated on 2023 May 15. As of that update, the firmware no longer works with the specific adapters I have. I am no longer able to use to guide to rebuild travel routers -- I instead use a backup image of my current travel router.**

Before plugging in the wireless adapter, check with `lsusb`. After plugging in the adapter, check again.

```
lsusb

ifconfig wlan1 up
```

```
ifconfig
```

Now, the external adapter (wlan1) is configured to be able to connect out to public wireless networks.

### Wireless file, part 2

Edit `/etc/config/wireless`.

```
vim /etc/config/wireless
```

Enable the wireless interface by **deleting** the line `radio1 > option disabled '0'`.

```
  option disabled '0'
```

Specify the default LAN as using `radio0`, which should be the built-in wireless adapter.

```
config wifi-iface 'default_radio0'
  option device 'radio0'
  option network 'lan'
  option mode 'ap'
  option ssid '$ssid'
  option encryption 'psk2'
  option key '$password'
```

Specify the `wifinet1` as using `radio1`, which should be the external wireless adapter.

```
config wifi-iface 'wifinet1'
  option device 'radio1'
  option mode 'sta'
  option network 'wwan'
  option ssid '$ssid'
  option encryption 'psk2'
  option key '$password'
```

Save & exit.

```
uci commit wireless

wifi
```

VPN connection
--------------

This is how to set up the VPN.

Head back to the command line via SSH.

```
opkg update
```

Confirm the following packages are installed:

```
opkg install openvpn-openssl luci-app-openvpn
```

Upload the VPN config file into the GUI, that should be it.

### Firewall

Using the GUI, navigate to the firewall section:

```
Network > Firewall
```

Go to the Zones section at the bottom, select the WAN (red) zone, and click 'edit':

```
Zones > WAN > edit
```

Go to Advanced Settings and check the `tunX` interface associated with the VPN connection (likely, `tun0`).

### Kill-switch

Using the GUI, navigate to the OpenVPN section.

```
VPN > OpenVPN
```

Enable the VPN connection (likely named 'client') to enable at startup. If the travel router is connecting to a network it has previously connected to, then an enabled OpenVPN service should give internet access on a fresh boot.

Now, navigate to the firewall section:

```
Network > Firewall
```

Go to the Zones section at the bottom, select the LAN (green) zone, and click 'edit':

```
Zones > LAN > edit
```

Go down to `Allow forward *to* destination zones` and remove the WAN zone. Save.

Add a new Zone by navigating below the Zones list and clicking the Add button.

Select:

- Masquerading
- MSS clamping
- Allow forward *from* source zones: LAN

Now, go to Advanced Settings, select `Covered Interfaces`, and *type* `tun+` into the entry field. Press 'enter'. Save & apply.

From [OpenWrt guide](https://openwrt.org/docs/guide-user/services/vpn/openvpn/client-luci): "tun+ is a regex that allows this rule to work with up to 10 tun interfaces (i.e. 10 VPNs) at the same time, if you have more, you need to adjust it. Then on the bottom of the page, click on Save and Apply button as usual to confirm and save your changes."

### Disable VPN kill-switch

Using the GUI, navigate to the firewall section:

```
Network > Firewall
```

Go to the Zones section at the bottom, select the LAN (green) zone, and click 'edit':

```
Zones > LAN > edit
```

Go down to `Allow forward *to* destination zones` and check the WAN zone. Save. Save & apply.

Using the Travel Router
-----------------------

### Troubleshooting

When using the travel router with the kill-switch VPN setup, any time the router connects to a new WAN, the OpenVPN server needs to be restarted.

In addition, when configuring the travel router to work as a VPN client to a VPN server with user credentials, the `auth-user-pass` should be included. It should also include the path to the file with the credentials, as is shown.

```
auth-user-pass /etc/openvpn/client.auth
```

References
----------

- [YouTube - NetworkChuck - Super Secure Raspberry Pi Router (Wireless VPN Travel Router)](https://www.youtube.com/watch?v=jlHWnKVpygw)
- [OpenWrt documentation - OpenVPN client using LuCI](https://openwrt.org/docs/guide-user/services/vpn/openvpn/client-luci)
    - Reference for VPN killswitch
