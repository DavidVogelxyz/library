# Installing Arch Linux, fast edition

## Introduction

This guide will work for both Arch and Artix Linux. Any differences between the two will be specified.

One difference worth noting is how to enable wireless internet.
- The Artix ISO uses the connection manager `connmanctl`. The commands are as follows:

```
connmanctl technologies
connmanctl enable wifi
connmanctl services
connmanctl
connmanctl> agent on
connmanctl> connect wifi_*  # connect to the correct wireless network; enter passphrase
connmanctl> quit
```

## Table of Contents
- [Introduction](#introduction)
- [Partitioning the storage drive](#partitioning-the-storage-drive)
- [Encrypting the root partition](#encrypting-the-root-partition)
- [Creating file systems](#creating-file-systems)
- [Mounting file systems](#mounting-file-systems)
- [Installing the operating system](#installing-the-operating-system)
- [Setting up the chroot environment](#setting-up-the-chroot-environment)
- [Making some necessary adjustments](#making-some-necessary-adjustments)
- [Kernel initialization](#kernel-initialization)
- [Bootloader](#bootloader)

## Partitioning the storage drive

Identify whether the computer is UEFI-compatible or not.

```
mount | grep efivars
```

or:

```
ls /sys/firmware/efi/efivars
```

Decide whether to install to a UEFI system or a legacy boot (BIOS) system.

On an Arch or Artix ISO, the shell should already be root. Otherwise, enter:

```
sudo -i
```

Use `lsblk` to identify the target drive (ex. sdx):

NB: For newer computers and laptops, the internal drive may look like `/dev/nvme0n1` and not `/dev/sda1`. While the guide uses `$sdxn` as a placeholder, users should substitute the correct drive.

### Partitioning with `fdisk`

Use `fdisk` to partition the drive:

```
fdisk /dev/$sdx
```

The following commands are useful in the order that they appear.

- d
: Deletes a partition. May have to be used numerous times. Can use numerals to specify which partition to delete.
- p
: Prints partition information. Good for confirming no partitions exist.
- g/o
: Creates a partition scheme. `G` is for GPT schemes, `o` is for DOS schemes (MBR).
  - If **UEFI**, use **GPT**!
  - If **legacy boot (BIOS)**, use **MBR**!
- n
: New partition (boot).
  - Create first partition, of size "+1G".
- t
: add Type information to a partition.
  - If using UEFI, or if an EFI parition needs to be created, use the `t` command to give it type `1`.
- n
: New partition (root).
  - Create second partition, of a size that uses the rest of the space on the storage drive.
- w
: Writes the partitions to the drive.

Use `lsblk` to confirm the changes made with `fdisk`.

## Encrypting the root partition

Also known as full disk encryption (FDE).

Use `cryptsetup` to encrypt a volume within the root partition. Then, unlock the encrypted volume.

```
cryptsetup luksFormat /dev/$sdx2

cryptsetup open /dev/$sdx2 <$LVM_NAME>
```

Use `lsblk` to confirm the changes made with `cryptsetup`. Also, use `ls /dev/mapper` to see the encrypted volume that was created.

```
ls /dev/mapper
```

## Creating file systems

### Swap partitions

***NB: I only use swap partitions on computers that run with batteries (laptops, etc). Otherwise, skip to [Making File Systems](#making-file-systems).***

Swap partitions are related to suspending computer activity. Most systems support suspending to RAM, but this requires the RAM to have power to save state. On a laptop, this will drain battery life. A better way to suspend activity is to use a swap partition.

Swap partitions work by suspending activity to the disk, rather than RAM. This allows the system to power off 100%, which effectively provides a more durable save state. However, a swap partition needs to be created first.

In general, the swap partition should be as large as the RAM space is. For general purposes, the guide has the value set at 4GB. Change this number to whatever is needed by the system.

First, initialize a physical volume. Users using full disk encryption (FDE) will create using `/dev/mapper/<$LVM_NAME>`. Users without FDE will create using `/dev/$sdx2`.

```
pvcreate /dev/mapper/<$LVM_NAME>
```

Next, create a volume group from that physical volume. Users using full disk encryption (FDE) will create using `/dev/mapper/<$LVM_NAME>`. Users without FDE will create using `/dev/$sdx2`.

`vgartix` can be any name for a volume group.

```
vgcreate vgartix /dev/mapper/<$LVM_NAME>
```

Create a logical volume for the swap partition. The command below assumes a RAM capacity of 4GB -- if different, change it!

```
lvcreate -L 4G -n swap_1 vgartix
```

Create a logical volume for the root file system, using all available space remaining.

```
lvcreate -l 100%FREE -n root vgartix
```

With the swap partition created, continue on.


### Making file systems

Create a file system for the boot partition.

```
mkfs.fat -F32 /dev/$sdx1
```

Create a file system for the root partition. Users using full disk encryption (FDE) will make the ext4 file system on `/dev/mapper/<$LVM_NAME>`. Users without FDE will make the file system on `/dev/$sdx2`. Users using a swap partition will make on `/dev/mapper/vgartix-root`.

```
mkfs.ext4 /dev/mapper/<$LVM_NAME>
```

If a swap partition was created, create a file system for it using the following command:

```
mkswap /dev/mapper/vgartix-swap_1
```

## Mounting file systems

```
mount /dev/mapper/<$LVM_NAME> /mnt

mkdir -pv /mnt/boot

mount /dev/$sdx1 /mnt/boot
```

## Installing the operating system

Edit the `pacman` mirror list with `vim` so a closer mirror is at the top of the list.

```
vim /etc/pacman.d/mirrorlist
```

Now, there are four different versions of the following command.

1. Primary variable is whether the install is for Arch or Artix Linux.
1. The second variable is whether the install is for legacy boot (BIOS) or UEFI.

### Arch Linux (BIOS)

Run the `pacstrap` command:

```
pacstrap -K -i /mnt base base-devel linux linux-firmware cryptsetup lvm2 grub networkmanager dhcpcd openssh neovim vim
```

### Artix Linux (BIOS)

Run the `basestrap` command:

```
basestrap -i /mnt base base-devel linux linux-firmware runit elogind-runit cryptsetup lvm2 lvm2-runit grub networkmanager networkmanager-runit neovim vim
```

### For UEFI

Add the `efibootmgr` package at the end of the command. The following is the command for Arch:

```
pacstrap -K -i /mnt base base-devel linux linux-firmware cryptsetup lvm2 grub networkmanager dhcpcd openssh neovim vim efibootmgr
```

**OR**, for Artix:

```
basestrap -i /mnt base base-devel linux linux-firmware runit elogind-runit cryptsetup lvm2 lvm2-runit grub networkmanager networkmanager-runit neovim vim efibootmgr
```

## Setting up the *chroot* environment

Use the `lsblk -f` command to check the output, then add the output to the `/mnt/etc/default/grub` file:

```
lsblk -f >> /mnt/etc/default/grub
```

### Arch Linux

Use the `genfstab -U /mnt` command to check the output, then add the output to the `/mnt/etc/fstab` file:

```
genfstab -U /mnt >> /mnt/etc/fstab
```

Change into the chroot environment:

```
arch-chroot /mnt bash
```

### Artix Linux

Use the `fstabgen -U /mnt` command to check the output, then add the output to the `/mnt/etc/fstab` file:

```
fstabgen -U /mnt >> /mnt/etc/fstab
```

Change into the chroot environment:

```
artix-chroot /mnt bash
```

## Making some necessary adjustments

### Date, time, and locale

#### Time zone

`America/New_York` is the same as `US/Eastern`.

```
ln -s /usr/share/zoneinfo/US/Eastern /etc/localtime

ls -l /etc/localtime
```

#### Time

```
hwclock --systohc
```

#### Locale

Add two lines to `/etc/locale.conf`:

```
export LANG="en_US.UTF-8"
export LC_COLLATE="C"
```

Uncomment lines in `/etc/locale.gen`, then run `locale-gen`:

```
locale-gen
```

### Fix the `fstab` file

*NB: Because Arch and Artix Linux make use of genfstab/fstabgen (respectively), the only "fixing" that would necessary would be for a **swap partition**.*

Add a line for the swap partition. It should look like the following:

```
UUID=<UUID_swap> none swap defaults 0 0
```

After editing the file, run `cat` on the file to confirm the changes.

```
cat /etc/fstab
```

### Host names

New hostname:

```
echo "$HOST" > /etc/hostname

cat /etc/hostname
```

Add three lines to `nvim /etc/hosts`:

```
127.0.0.1	localhost
::1	    	localhost
127.0.1.1	$HOST.$localdomain $HOST
```

### Enable networking

#### Arch Linux (systemd)

Three simple commands:

```
systemctl enable systemd-networkd

systemctl enable dhcpcd

systemctl enable sshd
```

#### Artix Linux (runit)

One simple command:

```
ln -s /etc/runit/sv/NetworkManager /etc/runit/runsvdir/current
```

### Create new main user

```
passwd

useradd -G wheel -m $USER

passwd $USER

nvim /etc/sudoers
```

If you want the ability for storage decryption to auto-login to main user, edit some file. The file location for a `runit` system is `/etc/runit/sv/agetty-tty1/conf`:

On `runit`, you would add:

```
--autologin $USER
```

## Kernel initialization

Open `/etc/mkinitcpio.conf` using `neovim`.

In the "hooks" section, add the following two modules (encrypt and lvm2) between the block and filesystems modules, like so:

```
block encrypt lvm2 filesystems
```

Push the updates with:

```
mkinitcpio -p linux
```

## Bootloader

Edit `/etc/default/grub` using `neovim`.

Go to the bottom of the document and clean up the appended lines by commenting them out, removing the superfluous ones, and moving the useful lines up near the top of the file.

Edit the `GRUB_CMDLINE_LINUX_DEFAULT` line to include the following:

```
cryptdevice=UUID=<UUID_$sdx2>:<$LVM_NAME> root=UUID=<UUID_/dev/mapper/root>
```

In addition, while here, add to `GRUB_CMDLINE_LINUX_DEFAULT` the following items if the computer will be use to virtualize machines. For `amd_iommu` and `intel_iommu`, only add the one that corresponds to the CPU of the machine.

```
iommu=pt amd_iommu=on intel_iommu=on
```

If PCI IDs are known (in relation to GPU passthrough), those can be added to `GRUB_CMDLINE_LINUX_DEFAULT` as well.

```
vfio-pci.ids=abcd:wxyz,...
```

### If BIOS (legacy boot)

Run this:

```
grub-install --target=i386-pc /dev/$sdx
```

### If UEFI

Run this:

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

### After `grub-install`:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

# THE END
