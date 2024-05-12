# Installing Proxmox

## Storage solutions

### Web UI

Use to web UI to remove `local-lvm`.

`Datacenter > Storage > local-lvm > Remove`

Also, adjust `local` to be able to store VM images:

`Datacenter > Storage > local > Edit > Content: Include Disk Images and Containers`

### CLI

Use a console. Either SSH in, or use the following command:

`$PROXMOX > Shell`

Now, input the following three commands:

```
lvremove /dev/pve/data

lvresize -l +100%FREE /dev/pve/root

resize2fs /dev/mapper/pve-root
```

## Repositories

Edit the following:

```
vi /etc/apt/sources.list
```

Adjust lines to look like this:

[Only copy over the first 9 lines. Do not include the last six lines.](/proxmox/etc/apt/sources.list)

Comment out:

```
vi /etc/apt/sources.list.d/pve-enterprise.list
```

Comment out the source repository here, so the lines look like the last two lines of the following link:

[Check the last 2 lines of this sources.list file](/proxmox/etc/apt/sources.list)

Test it out:

```
apt update && apt install vim neovim
```

### Nala

#### Install `nala` and its dependencies

To confirm this is done correctly, run `apt update` again and check the number of updatable packages. Write it down.

Then, edit the following:

```
nvim /etc/apt/sources.list
```

Add the following lines:

[Copy lines 11 and 12 to the bottom of the Proxmox server's `sources.list`](/proxmox/etc/apt/sources.list)

Run `apt install nala` and it should install `nala` clean with no issue.

Immediately do the next steps under Preferences.

#### Preferences

Copy the following file into either `/etc/apt/preferences` or `/etc/apt/preferences.d/preferences`

[Preferences file](/proxmox/etc/apt/preferences)

Once that file is in place, run `nala update`. It should return the same number of updatable packages as before. However, when running the following command `nala install nala`, it should have no issues sourcing the packages.

#### Test it out

```
nala upgrade

nala install cryptsetup
```

## Check out `sshd`

```
nvim /etc/ssh/sshd_config`
```

## PCIe (GPU) passthrough

### How to set up GPU passthrough

Check out my guide on [GPU passthrough with Proxmox](/proxmox/proxmox-gpu-passthrough.md).

## Backups

```
cryptsetup open /dev/sda1 cryptstorage

mount /dev/mapper/cryptstorage /mnt

/var/lib/vz/images/

/etc/pve/qemu-server/

sha256sum /var/lib/vz/template/iso/virtio-win-0.1.229.iso
```

## VLAN Aware

## Upload ISOs

### Upload VirtIO drivers ISO before uploading Windows ISOs

## Link Aggregation (LAG) with Proxmox

## References

- [YouTube - NetworkChuck - Virtual Machines Pt. 2 (Proxmox install w/ Kali Linux)](https://www.youtube.com/watch?v=_u8qTN3cCnQ)
    - Original video that showed how to install Proxmox using the GUI
    - Specifically, the reference for "LVM resizing" commands
- [YouTube - Techno Tim - Before I do anything on Proxmox, I do this first...](https://www.youtube.com/watch?v=GoZaMgEgrHw)
    - Another video that showed how to essential Proxmox setup
    - Specifically, the reference for GPU passthrough on Proxmox
