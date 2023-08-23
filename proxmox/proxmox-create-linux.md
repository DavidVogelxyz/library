# Configs for VMs

## Base Linux VM

- Graphics: Spice (qxl) (default)
- Machine: Default (i440fx)
- Storage: 50GB
- Sockets, cores: 1, 4
- Memory: 8192

After install, I reduce the CPU to 2 cores and RAM to 4096.

### Legacy boot

BIOS: Default (SeaBIOS)

### UEFI

BIOS: OVMF (UEFI)

UEFI has a slightly different first boot. As [this article](https://wiki.archlinux.org/title/Proxmox/Install_Arch_Linux_as_a_guest) from the Arch wiki describes, go into BIOS at start (press ESC) and disable secure boot, then boot from CD drive manually.
