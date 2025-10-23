How to migrate an existing Proxmox server to a new VLAN
=======================================================

This guide will assist the reader in creating a new VLAN (on pfSense) and migrating an existing Proxmox server to the new subnet.

Table of contents
-----------------

- [Introduction](#introduction)
- [Creating a new VLAN on pfSense](#creating-a-new-vlan-on-pfsense)
- [Reconfiguring Proxmox to use the new VLAN](#reconfiguring-proxmox-to-use-the-new-vlan)
- [Configuring the switch](#configuring-the-switch)
- [Testing the change](#testing-the-change)

Introduction
------------

In order to do this correctly, some steps should be taken in the proper order.

Creating a new VLAN on pfSense
------------------------------

First, create the new VLAN in the `Interfaces > VLAN` section.

Then, use that new VLAN to create the new interface.

Next, make sure that the following three items are set up properly:

- DHCP server
- Firewall rules
- NAT rule

As an example, consider a VLAN that contains two VM servers: one for server VMs, and one for testing VMs. It may not be the best idea to have both on the same VLAN, so it's time to separate.

First, choose which VM server is staying, and which is migrating. Then, create a new VLAN for that VM server. Next, create the interface using that new VLAN. The DHCP server for the new VLAN should be nearly identical to the DHCP server settings of the old VLAN; the only difference would be the IP ranges. For the firewall rules, it's probably alright to just copy the entire list of firewall rules and migrate them. For the NAT rule, just copy the original VLAN's NAT rule and change the IP range.

Reconfiguring Proxmox to use the new VLAN
-----------------------------------------

Next, change the IP address on the Proxmox. It is important that this step is done before the switch, as changing the rules on the switch will knock out the connectivity.

There are two files which will *definitely* require a change. Those are the `/etc/network/interfaces` and the `/etc/hosts` files. For both of these files, change any reference to the old IP address to reflect the new IP address the Proxmox will have on the new VLAN.

Depending on network setup, the `/etc/resolv.conf` file may also need an edit. This will only be the case in a network where the Proxmox is connecting to a local DNS server. If the DNS servers are outside the network, no change is needed.

It may also be a good idea to change the password to something "less secure", as consoling into the Proxmox with a "difficult-to-type" password will compound the pain of any error.

At this point, shut down all VMs and power down the Proxmox.

Configuring the switch
----------------------

When the [pfSense](#pfsense) and [Proxmox](#proxmox) settings are all ready to go, it is time to change the switch settings. Depending on what switch is being used, there may be multiple places where the VLAN tag needs to be changed.

As an example, a household TP-Link switch may have two places. One is on the page where the user indicates the tagging and untagging, and the other is the PVID page, where the switch is told which port corresponds to which VLAN.

Testing the change
------------------
