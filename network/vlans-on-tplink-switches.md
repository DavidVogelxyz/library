# Configuring VLANs on TP-Link switches, connected to a pfSense router

NB: This guide is for use with TP-Link TL-SG1xxE managed switches. This guide is confirmed to work both with the 8 port (TL-SG108E) and the 5 port (TL-SG105E) TP-Link switches.

## Introduction

This guide was written because the documentation found [here](https://www.tp-link.com/us/support/faq/788/) failed to work without specific additional steps. In this write-up, those extra steps are outlined.

## Table of contents

- [Introduction](#introduction)
- [Configuring pfSense](#configuring-pfsense)
    - [Creating VLANs](#creating-vlans)
    - [Enabling VLAN interfaces](#enabling-vlan-interfaces)
    - [Configuring VLAN DHCP servers](#configuring-vlan-dhcp-servers)
    - [Configuring VLAN NAT rules](#configuring-vlan-nat-rules)
- [Configuring the switch](#configuring-the-switch)
    - [VLAN configuration](#vlan-configuration)
    - [PVID settings](#pvid-settings)
- [References](#references)

## Configuring pfSense

The first step is to create the VLANs, assign them to the correct interfaces, and confirm that all other necessary configuations have been set.

### Creating VLANs

To [create the VLANs using pfSense](https://nguvu.org/pfsense/pfsense-baseline-setup/#create%20vlans), log into the pfSense web UI dashboard and navigate to "Interfaces > Assignments > VLANs". Assign a parent interface (the port on the pfSense that the VLAN will exist on), a VLAN tag, and a description. When all VLANs have been setup, [this image](https://nguvu.org/images/170127-042-vlan-interfaces.png) can be used to confirm what's been done so far.

For additional confirmation, navigate to "Interfaces > Assignments" and review the interface and network port. The VLANs should appear similar to what is shown in [this image](https://nguvu.org/images/170127-043-interface-assignments.png).

In addition, while still within pfSense, confirm [the VLAN interface is enabled](#enabling-vlan-interfaces), the [DHCP server is working](#configuring-vlan-dhcp-servers), and that [NAT rules have been configured](#configuring-vlan-nat-rules).

### Enabling VLAN interfaces

To [enable the VLAN interface](https://nguvu.org/pfsense/pfsense-baseline-setup/#configure%20interface%20ip%20addresses), navigate to "Interfaces > 'VLAN'". Set the configuration type as "static IP" and assign an IPv4 address range (ex. 10.10.10.1/24). Also, change the description (name) for the VLAN to a `$VLAN_name` that easily identifies the VLAN. See [this image](https://nguvu.org/images/170127-044-interface-config-vl10_mgmt.png) for a visual aid.

Without this step, the VLAN's gateway won't exist, and no device will be able to connect to the VLAN.

### Configuring VLAN DHCP servers

To [configure the VLAN's DHCP server](https://nguvu.org/pfsense/pfsense-baseline-setup/#configure%20interface%20dhcp), navigate to "Services > DHCP Server". Choose the correct tab named after the VLAN, check "enable DHCP server on `$VLAN_name` interface", and configure a primary address pool for the DHCP server by selecting a starting ("from") and ending ("to") IP address. This will set up a range of IP addresses for the DHCP server to use by default. See [this image](https://nguvu.org/images/170127-045-dhcp-interface-config-vl10_mgmt.png) for a visual aid.

Without this step, any device that attempts to connect to the VLAN will not get an IP assignment. Unless an IP within the VLAN is manually assigned by the client itself, no device will be able to access the VLAN.

### Configuring VLAN NAT rules

To [set up NAT rules for the VLAN](https://nguvu.org/pfsense/pfsense-baseline-setup/#configure%20nat), navigate to "Firewall > NAT > Outbound". Confirm "outbound NAT mode" is set to "manual"; then, configure a rule for the VLAN that looks like [what's shown here](https://nguvu.org/images/170127-061-outbound-nat.png).

Without this step, devices that successfully connect to the VLAN and get an IP assignment will be able to interact with local devices, but will experience difficulties with connecting through the WAN.

With the VLAN configured correctly, the next step is to set up the switch so that it knows how to route the VLAN traffic.

## Configuring the switch

For the most part, the [guide found on TP-Link's website](https://www.tp-link.com/us/support/faq/788/) is useful. However, there are some slight configuration changes that must be made, or the switch will not route traffic correctly. The full setup can be found below.

Note: be sure to do both parts of this section, or the VLAN traffic will not be routed.

### VLAN configuration

First, log in to the switch's web UI dashboard and navigate to "VLAN > 802.1Q VLAN Configuration". Enable "802.1Q VLAN Configuration" and select apply.

Configurations are added by VLAN, not by port. Leave the default VLAN (ID # 1) as it is. Add any VLANs created on pfSense by entering the tag number and a name/description, and then configure it using the following guidelines:

- Select as "untagged" ports any ports that should "belong to the VLAN". Put another way, the "untagged ports" are the ones where, if a device is plugged into this port, it should be a member of the VLAN and have an IP address within the VLAN assigned.
- Select as "tagged" ports any port that connects back to the pfSense router.
- Any port that doesn't fit into these guidelines should remain as "not member".

The key difference between this guide and the TP-Link FAQ is that the FAQ does not make it clear that a pfSense router needs its wired connection to be set as "tagged".

[This image](https://static.tp-link.com/image009_1525420737886l.png) can be used as a reference; however, note the following points:

- In that image's example, there are two VLANs: ID # 2 and ID # 3.
    - Devices plugged into port # 2 are supposed to access VLAN # 2, which is why port # 2 is marked as "untagged" on VLAN # 2.
    - Devices plugged into port # 3 are supposed to access VLAN # 3, which is why port # 3 is marked as "untagged" on VLAN # 3.
- To match with this guide, port # 4 is the port that connects back to the pfSense, which is why port # 4 is marked as "tagged" for both VLANs # 2 and 3.
    - From personal experience, the default VLAN (1) appears to work without tagging port # 4 for VLAN ID # 1.
        - Put another way, ports # 1 and 5 should be able to access the default VLAN without tagging port # 4.
            - If this is not the case, and devices plugged into ports # 1 and 5 cannot access the default VLAN, try configuring port # 4 as "tagged" for the default VLAN, restart the router, and attempt the connection again.

### PVID settings

After configuring the VLAN on the switch, navigate to "VLAN > 802.1Q VLAN PVID Setting".

For this menu page, enter a VLAN ID in the "PVID" field, select all ports that should have connected devices be part of that VLAN, and click "apply".

When completed, the PVID table should show the a list of ports where:

- Any port that is part of a VLAN is labeled with the respective PVID.
- Any port that is not part of a VLAN is labeled with the default PVID (1).

[This image](https://static.tp-link.com/image010_1525420757235x.png) can be used as a reference.

Now, when a device is plugged into the corresponding switch port, it should be assigned an IP address within that VLAN.

## References

- [TP-Link's official FAQ for creating VLANs on these switches](https://www.tp-link.com/us/support/faq/788/)
- [Nguvu.org guide on installing and configuring pfSense](https://nguvu.org/pfsense/pfsense-baseline-setup/)
- [YouTube - Lawrence Systems - TP-Link Switch Setup](https://www.youtube.com/watch?v=5ohLAFHnOHg)
    - This video doesn't explicity resolve the issue, but it is the video where the misconfiguration became clear.
