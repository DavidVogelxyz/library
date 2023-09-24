# Installing Debian / Ubuntu, fast edition

## Introduction

This guide uses the `debootstrap` command to install a clean Debian/Ubuntu Server image. Any differences between the two will be specified.

NB: Use the Ubuntu Desktop image over the Ubuntu Server image to use `debootstrap`. True whether installing Debian or Ubuntu. Why?
- Can scroll output on the terminal screen.
- Can paste in commands from the guide.

## Table of Contents
- [Introduction](#introduction)
- [Partitioning the storage drive](#partitioning-the-storage-drive)
- [Encrypting the root partition](#encrypting-the-root-partition)
- [Creating file systems](#creating-file-systems)
- [Mounting file systems](#mounting-file-systems)
- [Installing the operating system](#installing-the-operating-system)
- [Entering the chroot environment](#entering-the-chroot-environment)
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

Enter root shell:

```
sudo -i
```

Use `lsblk` to identify the target drive (ex. sdx).

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

`vgdebian` can be any name for a volume group.

```
vgcreate vgdebian /dev/mapper/<$LVM_NAME>
```

Create a logical volume for the swap partition. The command below assumes a RAM capacity of 4GB -- if different, change it!

```
lvcreate -L 4G -n swap_1 vgdebian
```

Create a logical volume for the root file system, using all available space remaining.

```
lvcreate -l 100%FREE -n root vgdebian
```

With the swap partition created, continue on.


### Making file systems

Create a file system for the boot partition.

```
mkfs.fat -F32 /dev/$sdx1
```

Create a file system for the root partition. Users using full disk encryption (FDE) will make the ext4 file system on `/dev/mapper/<$LVM_NAME>`. Users without FDE will make the file system on `/dev/$sdx2`. Users using a swap partition will make on `/dev/mapper/vgdebian-root`.

```
mkfs.ext4 /dev/mapper/<$LVM_NAME>
```

If a swap partition was created, create a file system for it using the following command:

```
mkswap /dev/mapper/vgdebian-swap_1
```

## Mounting file systems

```
mount /dev/mapper/<$LVM_NAME> /mnt

mkdir /mnt/boot

mount /dev/$sdx1 /mnt/boot
```

## Installing the operating system

```
apt update && apt install debootstrap vim
```

As of writing, the current Debian release (12) is `bookworm`, and the previous release (11) was `bullseye`. The current Ubuntu release is `jammy`.

```
debootstrap $release /mnt
```

When `debootstrap` completes, run the following command:

```
for d in sys dev proc ; do mount --rbind /$d /mnt/$d && mount --make-rslave /mnt/$d ; done
```

Also, copy over the `/etc/resolv.conf` file to assist with DNS queries:

```
cp -v /etc/resolv.conf /mnt/etc/
```

### Package management

Set up the `/mnt/etc/apt/sources.list` depending on the operating system installed.

#### Sources.list on Debian

[Debian wiki](https://wiki.debian.org/SourcesList#Example_sources.list) has example Debian package lists. Copy one over to `/mnt/etc/apt/sources.list`.

```
vim /mnt/etc/apt/sources.list
```

#### Sources.list on Ubuntu

Copy over the installer's `sources.list` to the new installation.

```
cp -v /etc/apt/sources.list /mnt/etc/apt/sources.list
```

NB: Remember to remove `CD-ROM` and add a line for `jammy-backports` (with `universe` for all) for `nala` and `neovim`. *May not be necessary in future releases.*

```
vim /mnt/etc/apt/sources.list
```

## Entering the *chroot* environment

Change into the chroot environment:

```
chroot /mnt /bin/bash
```

### Setting up the *chroot* environment

In Arch, the *chroot* environment is set up outside. With Debian and Ubuntu, it's done on the inside.

First, `cat` the information on currently mounted drives into the `/etc/fstab` file:

```
cat /proc/mounts >> /etc/fstab
```

Use the `blkid | grep UUID` command to check the output, then add the output to the `/etc/fstab` file:

```
blkid | grep UUID >> /etc/fstab
```

Install `nala`, a wrapper for `apt`, to get a better view of the package manager.

```
apt update && apt install nala
```

Then, using `nala`, install vim and neovim. If on Debian, install the `locales` package.

```
nala upgrade && nala install vim neovim locales
```

### Date, time, and locale

#### Time zone

`America/New_York` is the same as `US/Eastern`.

```
dpkg-reconfigure tzdata

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

### Kernel edit ***(UBUNTU ONLY)***

Absolutely necessary to create this file ***BEFORE*** installing `linux-image-generic` on Ubuntu **ONLY**, or the whole install breaks.

```
nvim /etc/kernel-img.conf
```

Add these two lines:

```
do_symlinks=no
no_symlinks=yes
```

NB: If this file is added to a Debian system, the install will proceed as normal. However, on future kernel updates, the system will break.

### Install packages

Re-run an upgrade command to confirm all packages are up-to-date.

```
nala upgrade
```

Install the base Linux kernel along with `Network Manager`. Also, if using an Intel CPU, include the `intel-microcode` package here.

```
nala install linux-image-generic build-essential network-manager # intel-microcode
```

Confirm that some basic and essential packages are installed by installing them now. Also, if installing to an encrypted root partition, include the `cryptsetup-initramfs` package here.

```
nala install curl gpg htop lvm2 rsync ssh sudo # cryptsetup-initramfs
```

#### Should already be installed

```
nala install grub-pc
```

#### Only necessary for UEFI builds

```
nala install grub-efi
```

### Fix the `fstab` file

Fix the `/etc/fstab` file.

```
nvim /etc/fstab
```

To fix the file:

- Delete all lines between (& including) `sysfs` through `devpts`.
- Delete lines below (but ***not*** including) `tmpfs` until `blkid | grep UUID` output. Remove `dev/sr0`.
- Change `tmpfs` directory to `/tmp`.
- Replace all `/proc/mounts` instances of `/dev/*` with the UUIDs from `/proc/mounts` output.
- Remove the quotation marks surrounding the UUIDs.
- Then delete the lines below `tmpfs`.

Only for installs that include a swap partition:

- Add a line for the swap partition. It should look like the following:

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
127.0.0.1   localhost
::1         localhost
127.0.1.1	$HOST.$localdomain $HOST
```

### Enable networking

One simple command:

```
systemctl enable NetworkManager
```

On Ubuntu, a config file must be made:

```
nvim /etc/netplan/networkmanager.yaml
```

Add the following lines:

```
network:
 version: 2
 renderer: NetworkManager
```

### Create new main user

```
passwd

useradd -G sudo -m $USER

passwd $USER

chsh -s /bin/bash $USER

nvim /etc/sudoers
```

## Kernel initialization

Instead of editing `/etc/default/grub` as is done on Arch Linux, update the `/etc/crypttab`:

```
<$LVM_NAME>  <UUID_$sdx2>  none luks
```

Update RAM fs:

```
update-initramfs -u -k all
```

## Bootloader

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

Unlike on Arch Linux and similar distribution, this shouldn't be necessary, because no changes were put into effect. In those distributions, the decryption of the root partition is handled by `mkinitcpio`, which is why adjustments are made to `/etc/default/grub`. On Debian and Ubuntu, the kernel does not handle the decryption, and `/etc/crypttab` will instead guide the decryption process.

But, just to be sure, run it anyways.

```
update-grub
```

# THE END
