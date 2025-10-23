GRE tunnels between Linux machines
==================================

Introduction
------------

GRE (Generic Routing Encapsulation) is a proprietary protocol developed by Cisco for encapsulating and routing traffic. There are better ways to tunnel and send traffic over the internet; however, here is a simple guide for anyone who wants to work with this protocol.

Table of contents
-----------------

- [Introduction](#introduction)
- [Setting up a GRE tunnel using Linux computers](#setting-up-a-gre-tunnel-using-linux-computers)
    - [Commands for computer A](#commands-for-computer-a)
    - [Commands for computer B](#commands-for-computer-b)
    - [Wrapping up](#wrapping-up)
- [References](#references)

Setting up a GRE tunnel using Linux computers
---------------------------------------------

First, confirm that the GRE kernel module is loaded:

```
sudo modprobe ip_gre
```

Confirm that it's active with the following command:

```
lsmod | grep gre
```

Let's imagine an example where a GRE tunnel needs to be opened between Computer A (50.50.50.50) and Computer B (110.110.110.110).

Both computers need to open the tunnel for it to be active. To do this, Computer A ("50") needs to open a connection to Computer B ("110"), and Computer B ("110") needs to open its own connection to Computer A ("50").

### Commands for computer A

First, from Computer A ("50"), the tunnel needs to be opened to Computer B ("110"). Use the following command to add the tunnel interface:

```
sudo ip tunnel add gre$numtun mode gre remote $ipb local $ipa ttl 255
```

`$numtun` is the number of the tunnel interface being created, `$ipb` is the IP address of Computer B ("110"), and `$ipa` is the IP address of Computer A ("50").

In the previous command, a GRE interface was created. Now, set the GRE interface as "up":

```
sudo ip link set gre$numtun up
```

Now that the interface is up, add the first IP address in the range to the GRE interface device:

```
sudo ip addr add 10.10.$numtun.1/24 dev gre$numtun
```

If opening multiple tunnels, this IP range scheme can be helpful, as it assists with knowing which IP range corresponds to which GRE tunnel interface. If necessary, change the second octet (the second "10") to a different number, so as to avoid conflicts with local networks.

### Commands for computer B

These are essentially the same commands as Computer A; however, the first one has the `local` and `remote` IP switched.

Add the tunnel interface:

```
sudo ip tunnel add gre$numtun mode gre remote $ipa local $ipb ttl 255
```

As before, `$numtun` is the number of the tunnel interface being created, `$ipa` is the IP address of Computer A ("50"), and `$ipb` is the IP address of Computer B ("110").

Set the GRE interface as up:

```
sudo ip link set gre$numtun up
```

Add the second IP address in the range to the GRE interface device:

```
sudo ip addr add 10.10.$numtun.2/24 dev gre$numtun
```

As before, if opening multiple tunnels, this IP range scheme can be helpful. If necessary, change the second octet (the second "10") to a different number, so as to avoid conflicts with local networks.

Wrapping up
-----------

Now, both computers should be able to reach each other by pinging the other's IP address on the tunnel interface.

Computer B ("110") can ping Computer A ("50") with:

```
ping 10.10.$numtun.1
```

Computer A ("50") can ping Computer B ("110") with:

```
ping 10.10.$numtun.2
```

That's how to easily set up GRE tunnels between two Linux machines!

References
----------

- [Xmodulo - How to create a GRE tunnel on Linux](https://www.xmodulo.com/create-gre-tunnel-linux.html)
