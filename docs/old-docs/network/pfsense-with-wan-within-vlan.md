Configuring pfSense with a "WAN within a VLAN"
==============================================

Introduction
------------

NB: Throughout this guide, the terms "router" and "gateway" will be used interchangeably.

Some internet service providers (ISPs) will provide a local gateway that receives a public IP through DHCP, but within a VLAN. Therefore, connecting the WAN ethernet into a different gateway will fail to produce a DHCP lease for that gateway, if the gateway isn't configured to communticate with the VLAN. This guide provides instructions for working with a pfSense router to properly set up the VLAN for the WAN connection, such that a DHCP lease is provided and the WAN is accessible.

To reiterate, this document is written from the perspective of a user working with a pfSense router. If using a different router, then the steps will be different; however, the underlying principles should be nearly identical.

Table of contents
-----------------

- [Introduction](#Introduction)
- [Context](#Context)
- [Identifying the issue](#Identifying-the-issue)
- [Configuring a pfSense to work with a WAN using a VLAN](#Configuring-a-pfSense-to-work-with-a-WAN-using-a-VLAN)

Context
-------

Often, when an internet user becomes more experienced with networking, they may want to connect their own network hardware directly to an ISP, rather than using the hardware provided by the ISP. Sometimes, this is as easy as removing the WAN ethernet cable from the ISP's router and connecting it to personal hardware. Comcast is an example of an ISP that, in some cases, is configured to lease IPs to the end user using DHCP without a VLAN. In cases such as these, then the personal router should receive a DHCP lease and the WAN should be immediately accessible to the new gateway.

In other cases, the personal router will not receive a DHCP lease and the WAN will not be accessible to that gateway. This can occur for a variety of reasons. One example reason is that the gateway was assigned a static IP configuration. Cogent is an ISP that, in some cases, is configured to only connect a gateway that has been configured with the correct static IP assignment. In this case, the personal router needs to be set up with the same static IP configuration as the provided gateway; or, using Cogent as an example, the gateway needs to be configured with the static IP assignment provided when the network was installed.

However, there is a third possibility: the ISP is using VLANs in combination with DHCP. Therefore, a personal router *can* be connected to the WAN, and an IP *can* be provided via DHCP, but only if the gateway is VLAN-aware. In addition, the router needs also to be configured to communicate *with the correct VLAN* in order to receive a DHCP lease. This situation is the one that this guide intends to assist with resolving.

Identifying the issue
---------------------

The simplest way to discern whether or not the ISP-provided router is using a VLAN is to connect to that gateway's web dashboard and answer the following questions:

1. Is the original (ISP-provided) gateway connecting to the WAN using "DHCP" or a "static IP assignment"?
    - If the gateway is using a "static IP assignment", then the new (personal) router will need to use a static IP assignment that mirrors the one found on the original gateway.
    - If the gateway is using "DHCP", look around to see whether or not a VLAN is configured.
        - If those configurations **are** visible, confirm the use of a VLAN.
            - If the original gateway is **NOT** using a VLAN, **STOP HERE**. Configuring the new router should be easy.
            - If the original gateway **is** using a VLAN, then it should be possible to get the VLAN ID directly from the original gateway, as the VLAN ID needs to be set at the gateway level. **Record that VLAN ID for later**.
        - If those configurations **are not** visible, then it's likely that the original gateway **is not** using a VLAN. Again, the VLAN ID needs to be set at the gateway level; so, if it's not visible, there's a good chance it wasn't set up to use a VLAN.
        - If the original gateway is using a VLAN, and it's not possible to get the VLAN ID from the gateway itself, continue.
1. After connecting the WAN to the new (personal) router, did the new gateway receive an IP lease?
    - If the new router received an IP address from upstream, then all is good!
    - If not, it is possible that the ISP is using a VLAN as part of their network setup.
        - At this step, if it wasn't possible to confirm with the original gateway, then it may be worthwhile to perform a search to find out if anyone else has confirmed the VLAN situation for that particular ISP. Once the VLAN has been identified, **record the VLAN ID for later**.

At this point, there should be confirmation that the original gateway is using a VLAN along with DHCP, and the VLAN ID should have been identified.

Configuring a pfSense to work with a WAN using a VLAN
-----------------------------------------------------

With the VLAN ID in hand, log in to the pfSense's web dashboard.

### Creating the VLAN

First, the VLAN needs to be configured. Navigate to `Interfaces > Assignments`, then `VLANs`, and create a new VLAN.

- Set the `Parent Interface` as `WAN`.
- Set the `VLAN Tag` as the VLAN ID that the ISP is using.
    - Hopefully, the ISP's VLAN ID is not already in use. If so, the network will need to be reconfigured, so that the VLAN ID is available for use with the WAN.
- Set the `VLAN Priority` as 0.
    - Alternatively, leave it blank.
- Set the `Description` as the name of the interface.
    - Example: `<NAME_OF_ISP>`

### Configuring the VLAN interface

Next, configure a new interface. To do this, navigate to `Interfaces > Assignments` and create a new interface.

- At the bottom of the list, there should be the line item `Available network ports` using the `Add` button.
- Select the VLAN that was just created.
    - The option should look similar to the following: `VLAN <ID> on WAN`
- Click `Add`.

Now, configure the new interface to receive an IP address via DHCP. Navigate to `Interfaces > Assignment`. Click on the link, under the column `Interface`, for the newly assigned interface. While the network port should contain `VLAN <ID> on WAN`, the name under the `Interface` column will likely have a name like `OPT_`. Configure the new interface as follows:

- The interface should be `enabled`.
- The `Description` should be the name of the ISP.
    - Example: `<NAME_OF_ISP>`
    - The `Description` field will take the place of the `Interface` field on the `Interfaces > Assignment` page.
- The `IPv4 configuration type` should be DHCP.
- All other settings can mirror the settings of the original `WAN` interface.

### Configuring the Network Address Translation (NAT)

The last thing that needs to be done is to confirm that the Network Address Translation (NAT) rules are configured to work with this new interface. Navigate to `Firewall > NAT > Outbound`.

- If the NAT rules have already been configured, it should be relatively simple to modify the rules.
- Go through each NAT rule.
    - The `Interface` should currently be set to `WAN`.
    - The `Interface` now needs to be set to `<NAME_OF_ISP>`, or whatever `Description` was set in the previous step.
    - Every other setting can be left as is.

With all of this done, the WAN ethernet cable can now be connected to the pfSense router. Navigate to the main dashboard of the pfSense's web interface. If everything is working correctly, the `Interface` table should now show the `<NAME_OF_ISP>` interface as having received an IP assignment. Confirm the WAN is accessible by navigating to a webpage outside of the LAN -- the page should load as intended.

### Configuring the firewall

As an additional step, any firewall rules that were originally set for the `WAN` should also be set for the new `VLAN`.

---

Congrats on setting up a personal router to connect directly to the ISP, through a VLAN, without using the ISP's equipment!
