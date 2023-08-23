# Installing Arch Linux, fast edition

## Introduction

This guide will work for both Arch and Artix Linux. Any differences between the two will be specified.

## Partitioning the storage drive

Identify whether the computer is UEFI-compatible or not.

```
ls /sys/firmware/efi/efivars
```

Decide whether to install to a UEFI system or a legacy boot (BIOS) system.

On an Arch or Artix ISO, the shell should already be root. Otherwise, enter:

```
sudo -i
```

Use `lsblk` to identify the target drive (ex. sdx):

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

Use `cryptsetup` to encrypt a volume within the root partition. Then, unlock the encrypted volume.

```
cryptsetup luksFormat /dev/$sdx2

cryptsetup open /dev/$sdx2 $LVM_NAME
```

Use `lsblk` to confirm the changes made with `cryptsetup`.

## Creating file systems

```
ls /dev/mapper

mkfs.fat -F32 /dev/$sdx1

mkfs.ext4 /dev/mapper/$LVM_NAME
```

## Mount file systems

```
mount /dev/mapper/$LVM_NAME /mnt

mkdir /mnt/boot

mount /dev/$sdx1 /mnt/boot
```

## Installing the Operating System

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

Add the `efibootmgr` package at the end of the command:

```
pacstrap -K -i /mnt base base-devel linux linux-firmware cryptsetup lvm2 grub networkmanager dhcpcd openssh neovim vim efibootmgr
```

**OR**

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

`America/New_York` is the same as `US/Eastern`

```
ln -s /usr/share/zoneinfo/America/New_York /etc/localtime

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

### Host names

New hostname:

```
echo "$HOST" > /etc/hostname

cat /etc/hostname
```

Add three lines to `nvim /etc/hosts`:

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	$HOST.localdomain $HOST
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

## Kernel init

Add one section to `/etc/mkinitcpio.conf`:

```
block encrypt lvm2 filesystems
```

Push the updates with:

```
mkinitcpio -p linux
```

## Bootloader

Adjust `/etc/default/grub`

Add the following to a very specific spot:

```
cryptdevice=UUID=[UUID_$sdx2]:cryptvolume root=UUID=[UUID_/dev/mapper/$LVM_NAME]
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
