# Connecting a QEMU VM to the local network

## Introduction

This guide will assist the user in connecting a QEMU VM to the local network.

By default, QEMU makes use of a virtual bridge that uses NAT. Therefore, connections leaving that virtualized network will appear to be coming from the host machine. One of the limitations of this setup is that any firewall permissions granted to the host will be extended to the VM.

In contrast, this guide will explain how to set up a new bridge connection that doesn't use NAT. Any VM that uses this bridge will instead be assigned an IP address on the local subnet, as if it was an entirely different machine. Therefore, specific firewall rules can be created to handle that VM that are separate from the host machine.

It may take a couple of attempts to get this to work correctly; but, once setup this way, it should automatically work on startup going forward.

Additional notes:

- The wired interfaces need to exist, but not be connected
    - IPv4 config can remain `automatic`
    - but, `automatically connect` should be disabled
    - when setting this up, the ethernet interface must be disabled so it can become a slave
- then, run the commands:
    - the inflection point occurs when running `ip link show master br0`:
        - if the `bridge-slave` doesn't show up, then restart from the beginning
            - delete the `bridge-slave` and the bridge, and try again
        - if all goes well, the bridge should come up without needing to run `sudo ip link set br0 up`
- then, set all wired interfaces to disable `automatically connect`

## How-to

Show connections in `nmcli`:

```
sudo nmcli con show
```

Show bridge connections using `brctl`:

```
sudo brctl show
```

Add a new bridge connection in `nmcli`:

```
sudo nmcli con add type bridge ifname br0
```

Create a new `bridge-slave` connection, linking Ethernet interface `eth0` to the previously created bridge interface `br0`:

```
sudo nmcli con add type bridge-slave ifname eth0 master br0
```

Show bridge connections using `ip link`:

```
ip link show type bridge
```

Show the connections to which `br0` is the master:

```
ip link show master br0
```

If `br0` is down, set it as `up` with:

```
sudo ip link set br0 up
```
