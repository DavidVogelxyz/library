# Installing QEMU on Arch Linux

Obviously, this works for Artix Linux too. Currently written with a focus on Runit.

## Instructions

Run:

```
alias pacman="p"
p -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat libguestfs
```

### Runit

Run:

```
p -S libvirt-runit
sudo usermod -aG libvirt $USER
```

Edit the following file:

```
sudo vim /etc/libvirt/libvirtd.conf
```

Make the following edits:

- Change `unix_sock_group` to equal `libvirt`
- Change `unix_sock_ro_perms` to equal `0777`
- Change `unix_sock_rw_perms` to equal `0770`

Run:

```
sudo ln -s /etc/runit/sv/libvirtd /etc/runit/runsvdir/current
sudo ln -s /etc/runit/sv/virtlogd /etc/runit/runsvdir/current
sudo ln -s /etc/runit/sv/virtlockd /etc/runit/runsvdir/current
sv restart libvirtd
sudo sv restart libvirtd
sv status libvirtd
```
