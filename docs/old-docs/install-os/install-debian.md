Installing Debian / Ubuntu, fast edition
========================================

Introduction
------------

This guide uses the `debootstrap` command to install a clean Debian/Ubuntu OS image. This guide can and should be followed regardless of whether the install is for a desktop or server installation. Any differences between the Debian and Ubuntu installs will be specified.

**NB:** Use an Ubuntu 24 (Noble) Desktop image over the Ubuntu Server image to perform the install. This guideline should be followed whether installing Debian or Ubuntu.

Why?

- By using the Ubuntu desktop GUI:
    - it is possible to scroll output in the terminal.
    - commands from the guide can be pasted into the terminal.

Table of Contents
-----------------

- [Introduction](#introduction)
- [Partitioning the storage drive](#partitioning-the-storage-drive)
    - [Partitioning with fdisk](#partitioning-with-fdisk)
- [Encrypting the root partition](#encrypting-the-root-partition)
- [Creating file systems](#creating-file-systems)
    - [Swap partitions](#swap-partitions)
    - [Making file systems](#making-file-systems)
- [Mounting file systems](#mounting-file-systems)
- [Installing the operating system](#installing-the-operating-system)
    - [Package management](#package-management)
        - [Sources list on Debian](#sources-list-on-debian)
        - [Sources list on Ubuntu](#sources-list-on-ubuntu)
- [Setting up the chroot environment](#setting-up-the-chroot-environment)
- [Making some necessary adjustments](#making-some-necessary-adjustments)
    - [Time zone](#time-zone)
    - [Time](#time)
    - [Locale](#locale)
    - [Kernel edit (UBUNTU ONLY)](#kernel-edit-ubuntu-only)
    - [Install packages](#install-packages)
    - [Fix the fstab file](#fix-the-fstab-file)
    - [Hostnames](#hostnames)
    - [Enable networking](#enable-networking)
    - [Create new main user](#create-new-main-user)
- [Kernel initialization](#kernel-initialization)
- [Bootloader](#bootloader)
    - [Bootloader - BIOS](#bootloader---bios)
    - [Bootloader - UEFI](#bootloader---uefi)
    - [Update GRUB](#update-grub)
- [References](#references)

Partitioning the storage drive
------------------------------

Identify whether the computer is UEFI-compatible or not using the following commands.

```
mount | grep efivars
```

or:

```
ls /sys/firmware/efi/efivars
```

If one of the above commands shows output, then both should display output. If there is an output, then install as a UEFI system. If there is no output, install as a legacy boot (BIOS) system.

If not already logged in as the root user, elevate privileges using the following command:

```
sudo -i
```

Use `lsblk` to identify the target drive (ex. "sd*" or "nvme*"):

```
lsblk
```

NB: For newer computers and laptops, the internal drive may be an NVMe drive (`/dev/nvme0n1`) and not a SATA drive (`/dev/sda1`). While the guide uses `$sdxn` as a placeholder, users should substitute the correct drive in the place of the variable. Where the placeholder variable appears in the guide, "x" represents the letter of the drive, and "n" represents the partition number. In the instances where `$sdx` appears, no partition value should be given -- only add a partition value when the "n" appears. Also, whenever a number is given in place of "n", that should be the correct partition number. Use best judgment.

### Partitioning with fdisk

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

Encrypting the root partition
-----------------------------

This step may be referred to in other parts of the guide as full disk encryption (FDE).

Use `cryptsetup` to encrypt a volume within the root partition.

```
cryptsetup luksFormat /dev/$sdx2
```

Then, unlock the encrypted volume. In place of `$LVM_NAME`, input a name for the LVM (ex. "cryptlvm").

```
cryptsetup open /dev/$sdx2 $LVM_NAME
```

Use `lsblk` to confirm the changes made with `cryptsetup`. Also, use `ls /dev/mapper` to see the encrypted volume that was created.

```
ls /dev/mapper
```

Creating file systems
---------------------

### Swap partitions

***NB: Use swap partitions only on computers that run with batteries (laptops, etc). Otherwise, skip to [Making file systems](#making-file-systems).***

Swap partitions are related to suspending computer activity. Most systems support suspending to RAM, but this requires the RAM to have power to save state. On a laptop, this will drain battery life. A better way to suspend activity is to use a swap partition.

Swap partitions work by suspending activity to the disk, rather than RAM. This allows the system to power off 100%, which effectively provides a more durable save state. However, a swap partition needs to be created first.

In general, the swap partition should be as large as the RAM space is. For general purposes, the guide has the value set at 4GB. Change this number to whatever is needed by the system.

First, initialize a physical volume. Users using full disk encryption (FDE) will create using `/dev/mapper/$LVM_NAME`. Users without FDE will create using `/dev/$sdx2`.

```
pvcreate /dev/mapper/$LVM_NAME
```

Next, create a volume group from that physical volume. Users using full disk encryption (FDE) will create using `/dev/mapper/$LVM_NAME`. Users without FDE will create using `/dev/$sdx2`.

`vgdebian` can be any name for a volume group.

```
vgcreate vgdebian /dev/mapper/$LVM_NAME
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

Create a file system for the root partition. Use the following bullet point list to decide which path on which to create the ext4 file system. Going in order, if a bullet point item is true, then stop there and use that path.

- Users using a swap partition will make the file system on `/dev/mapper/vgdebian-root`.
- Users using full disk encryption (FDE) will make the file system on `/dev/mapper/$LVM_NAME`.
- Users without FDE will make the file system on `/dev/$sdx2`.

```
mkfs.ext4 /dev/mapper/$LVM_NAME
```

If a swap partition was created, create a file system for it using the following command:

```
mkswap /dev/mapper/vgdebian-swap_1
```

Mounting file systems
---------------------

First, mount the root partition to the `/mnt` directory. This partition should have the same path as the partition on which the ext4 file system was made.

```
mount /dev/mapper/$LVM_NAME /mnt
```

Next, create the `/mnt/boot` directory with the following command:

```
mkdir -pv /mnt/boot
```

After creating the directory, mount the boot partition to that directory:

```
mount /dev/$sdx1 /mnt/boot
```

Installing the operating system
-------------------------------

To install an operating system to the drive: update packages, and then install both `debootstrap` and a text editor of choice (ex. `vim`).

```
apt update && apt install -y debootstrap vim
```

As of writing, the current Debian release (12) is `bookworm`, and the previous release (11) was `bullseye`. The current Ubuntu release (24) is `noble`, and the previous release (22) was `jammy`. Substitute the release of choice in place of `$release`.

```
debootstrap $release /mnt
```

When `debootstrap` completes, run the following command:

```
for d in sys dev proc ; do mount --rbind /$d /mnt/$d && mount --make-rslave /mnt/$d ; done
```

Also, copy over the `/etc/resolv.conf` file to assist with DNS queries.

**Note:** When installing Ubuntu 24 (Noble) [and potentially Debian 12 (bookworm)], this step doesn't appear to be necessary, as the `debootstrap` command seems to copy the `/etc/resolv.conf` file. Check both files to confirm, and copy the file only if it's not identical. That being said, even if they are identical, there's no concern about copying it over again anyway.

```
cp -v /etc/resolv.conf /mnt/etc/
```

### Package management

Set up the `/mnt/etc/apt/sources.list` depending on the operating system installed.

#### Sources list on Debian

[Debian wiki](https://wiki.debian.org/SourcesList#Example_sources.list) has example Debian package lists. Copy one over to `/mnt/etc/apt/sources.list`.

```
vim /mnt/etc/apt/sources.list
```

#### Sources list on Ubuntu

**Note:** This step only appears to be necessary on Ubuntu 22 (Jammy), as a recent `debootstrap` of Ubuntu 24 (Noble) already had a working `sources.list` file in `/etc/apt`. When installing Ubuntu 24 (Noble), the only step required is to edit the `sources.list` file to contain `universe` at the end of the entry. Enabling the `universe` repository will allow `nala` and `stow` to be installed in future steps.

If installing Ubuntu 22 (Jammy), copy over the installer's `sources.list` to the new installation.

```
cp -v /etc/apt/sources.list /mnt/etc/apt/sources.list
```

NB: Remember to remove the `CD-ROM` line, and add a line for `jammy-backports` (with `universe` for all) for `nala`.

```
vim /mnt/etc/apt/sources.list
```

Setting up the *chroot* environment
-----------------------------------

Change into the *chroot* environment:

```
chroot /mnt /bin/bash
```

NB: In order to save time and effort, there is a script called `debian-setup` that can be used to power through the remainder of the guide. It can be found [here](https://github.com/DavidVogelxyz/debian-setup).

First, `cat` the information on currently mounted drives into the `/etc/fstab` file:

```
cat /proc/mounts >> /etc/fstab
```

Append the output of the `blkid | grep UUID` command to the `/etc/fstab` file:

```
blkid | grep UUID >> /etc/fstab
```

Install `nala`, a wrapper for `apt`, to get a better view of the package manager.

```
apt update && apt install -y nala
```

Then, using `nala`, install `vim`. If on Debian, also install the `locales` package.

```
nala upgrade && nala install -y vim locales
```

Making some necessary adjustments
---------------------------------

### Time zone

On Debian-based systems, setting the time zone is easy to do through the use of the `dpkg-reconfigure` command:

```
dpkg-reconfigure tzdata
```

NB: `America/New_York` is the same as `US/Eastern`.

Confirm that the time zone was configured correctly with the following command:

```
ls -l /etc/localtime
```

### Time

Sync the system clock to the hardware clock using the following command:

```
hwclock --systohc
```

### Locale

Using a text editor, create the "/etc/locale.conf" and add the following two lines:

```
export LANG="en_US.UTF-8"
export LC_COLLATE="C"
```

Uncomment lines in "/etc/locale.gen" that pertain to the correct locale (ex. en_US*), then run `locale-gen`:

```
locale-gen
```

### Kernel edit ***(UBUNTU ONLY)***

If, ***and only if***, Ubuntu is being installed, it is absolutely necessary to create "/etc/kernel-img.conf" file ***BEFORE*** installing `linux-image-generic` (or the whole install breaks).

```
vim /etc/kernel-img.conf
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

Install the base Linux kernel along with `network-manager`. Also, if using an Intel CPU, include the `intel-microcode` package here (note that `intel-microcode` requires the `non-free-firmware` repository to be enabled).

```
nala install -y linux-image-generic build-essential network-manager # intel-microcode
```

Confirm that some basic and essential packages are installed by installing them now. Also, if installing to an encrypted root partition, include the `cryptsetup-initramfs` package here.

```
nala install -y curl git gpg htop lvm2 rsync ssh stow sudo # cryptsetup-initramfs
```

If installing to a BIOS system, the following package should already be installed. However, just to confirm, attempt to install it anyway:

```
nala install -y grub-pc
```

If, ***and only if***, installing to a UEFI system, install the following package. Installing `grub-efi` will remove the `grub-pc` package if it is already installed on the system.

```
nala install -y grub-efi
```

If the install was set up with FDE, append the output of the `blkid | grep UUID` command to the `/etc/crypttab` file:

```
blkid | grep UUID >> /etc/crypttab
```

### Fix the fstab file

Fix the `/etc/fstab` file.

```
vim /etc/fstab
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
UUID=$UUID_swap none swap defaults 0 0
```

After editing the file, run `cat` on the file to confirm the changes.

```
cat /etc/fstab
```

### Hostnames

Create a new hostname for the install by either using `echo` and ">" to overwrite the "/etc/hostname" file with the output. Alternatively, use a text editor to edit the file.

```
echo "$HOST" > /etc/hostname
```

To confirm the changes, use `cat` on the file:

```
cat /etc/hostname
```

Add three lines to "/etc/hosts":

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   $HOST $HOST.$localdomain
```

### Enable networking

On Debian, enabling networking is as simple as running one command:

```
systemctl enable NetworkManager
```

On Ubuntu, in addition to enabling `networkmanager`, the "/etc/netplan/networkmanager.yaml" config file must be created:

Add the following lines to it:

```
network:
 version: 2
 renderer: NetworkManager
```

### Create new main user

First, set a new password for the root user with the following command:

```
passwd
```

Next, create a new user with the `useradd` command. Add the user to the "sudo" group with `-G sudo`, and use `-m` to give that user a home directory. The `-s /bin/bash` is also good, because Debian tends to default the user to the `sh` shell, which has far less features than the standard `bash` shell most users are used to.

```
useradd -G sudo -s /bin/bash -m $USER
```

Once the user has been created, set the user's password with the following command:

```
passwd $USER
```

While it's not a requirement on Debian-based systems, it's good practice to confirm that users in the "sudo" group are allowed to run root-level commands with the `sudo` command. Near the bottom of the "/etc/sudoers" file, confirm that the command that enables users of group "sudo" to run "any command" is uncommented.

```
vim /etc/sudoers
```

Kernel initialization
---------------------

On a system using FDE, it is a requirement to update the `/etc/crypttab` file, so the system knows to request a password to unlock the encrypted root drive:

```
$LVM_NAME UUID=$UUID_sdx2 none luks
```

As before, `$LVM_NAME` is the name given to the cryptdevice, and `$UUID_sdx2` is the UUID of the block device that was encrypted.

Whether using FDE or not, update the inital RAM file system with the following command:

```
update-initramfs -u -k all
```

Bootloader
----------

### Bootloader - BIOS

If installing on a BIOS system, run the following command:

```
grub-install --target=i386-pc /dev/$sdx
```

### Bootloader - UEFI

If installing on a UEFI system, run the following command:

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

### Update GRUB

Unlike on Arch Linux and similar distribution, this shouldn't be necessary, because no changes were put into effect. In those distributions, the decryption of the root partition is handled by `mkinitcpio`, which is why adjustments are made to `/etc/default/grub`. On Debian and Ubuntu, the kernel does not handle the decryption, and `/etc/crypttab` will instead guide the decryption process.

But, just to be sure, run the following command anyways:

```
update-grub
```

---

Congrats on installing an Debian-based Linux system!

References
----------

- [Youtube - Animortis Fortress - Ubuntu 22.10 Debootstrap Installation](https://www.youtube.com/watch?v=UumkGuoy0tk)
    - The YouTube video the demonstrated how to install Debian using `debootstrap`
- [Ubuntu Help - Full Disk Encryption: How To (2019)](https://help.ubuntu.com/community/Full_Disk_Encryption_Howto_2019)
    - The original reference guide, used from 2020 until 2023
