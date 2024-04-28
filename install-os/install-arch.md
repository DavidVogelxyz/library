# Installing Arch / Artix Linux, fast edition

## Introduction

This guide will work for both Arch and Artix Linux. Any differences between the two will be specified.

## Table of Contents

- [Introduction](#introduction)
- [Enabling wireless internet on Artix Linux](#enabling-wireless-internet-on-artix-linux)
- [Partitioning the storage drive](#partitioning-the-storage-drive)
    - [Partitioning with fdisk](#partitioning-with-fdisk)
- [Encrypting the root partition](#encrypting-the-root-partition)
- [Creating file systems](#creating-file-systems)
    - [Swap partitions](#swap-partitions)
    - [Making file systems](#making-file-systems)
- [Mounting file systems](#mounting-file-systems)
- [Installing the operating system](#installing-the-operating-system)
    - [BIOS](#bios)
        - [BIOS - Arch Linux](#bios---arch-linux)
        - [BIOS - Artix Linux](#bios---artix-linux)
    - [UEFI](#uefi)
        - [UEFI - Arch Linux](#uefi---arch-linux)
        - [UEFI - Artix Linux](#uefi---artix-linux)
- [Setting up the chroot environment](#setting-up-the-chroot-environment)
- [Making some necessary adjustments](#making-some-necessary-adjustments)
    - [Time zone](#time-zone)
    - [Time](#time)
    - [Locale](#locale)
    - [Fix the fstab file](#fix-the-fstab-file)
    - [Hostnames](#hostnames)
    - [Enable networking](#enable-networking)
        - [Networking with systems using systemd](#networking-with-systems-using-systemd)
        - [Networking with systems using runit](#networking-with-systems-using-runit)
    - [Create new main user](#create-new-main-user)
- [Kernel initialization](#kernel-initialization)
- [Bootloader](#bootloader)
    - [Bootloader - BIOS](#bootloader---bios)
    - [Bootloader - UEFI](#bootloader---uefi)
    - [Update GRUB](#update-grub)

## Enabling wireless internet on Artix Linux

One difference worth noting is how to enable wireless internet.

The Artix ISO uses the connection manager `connmanctl`. The commands are as follows:

```
connmanctl technologies
connmanctl enable wifi
connmanctl services
connmanctl
```

After running `connmanctl`, a command-specific shell will open. Use the following commands (following the ">") to connect to a wireless network.

```
connmanctl> agent on
connmanctl> connect wifi_*  # connect to the correct wireless network; enter passphrase
connmanctl> quit
```

An additional item worth noting: if the computer that is receiving the Artix Linux install isn't connected via Ethernet, and requires `connmanctl`, it will also need `connmanctl` when first booted. Be sure to install `connmanctl` during the [Installing the operating system](#installing-the-operating-system) section of the guide, to avoid any connection issues after install.

## Partitioning the storage drive

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

## Encrypting the root partition

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

## Creating file systems

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

`vgartix` can be any name for a volume group.

```
vgcreate vgartix /dev/mapper/$LVM_NAME
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

Create a file system for the root partition. Use the following bullet point list to decide which path on which to create the ext4 file system. Going in order, if a bullet point item is true, then stop there and use that path.

- Users using a swap partition will make the file system on `/dev/mapper/vgartix-root`.
- Users using full disk encryption (FDE) will make the file system on `/dev/mapper/$LVM_NAME`.
- Users without FDE will make the file system on `/dev/$sdx2`.

```
mkfs.ext4 /dev/mapper/$LVM_NAME
```

If a swap partition was created, create a file system for it using the following command:

```
mkswap /dev/mapper/vgartix-swap_1
```

## Mounting file systems

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

## Installing the operating system

Edit the `pacman` mirror list with a text editor of choice (ex. `vim`) so mirrors closer to the computer's physical location are at the top of the list.

```
vim /etc/pacman.d/mirrorlist
```

NB: During the next part, once the correct command is chosen and run, be sure to select `mkinitcpio` as the package to manage the initial RAM file system (initramfs).

Now, there are four different versions of the following command.

1. The primary variable is whether the install is for legacy boot (BIOS) or UEFI.
1. The secondary variable is whether the install is for Arch or Artix Linux.

### BIOS

#### BIOS - Arch Linux

Run the `pacstrap` command:

```
pacstrap -K -i /mnt base base-devel linux linux-firmware cryptsetup lvm2 grub networkmanager dhcpcd openssh neovim vim
```

#### BIOS - Artix Linux

Run the `basestrap` command:

```
basestrap -i /mnt base base-devel linux linux-firmware runit elogind-runit cryptsetup lvm2 lvm2-runit grub networkmanager networkmanager-runit neovim vim
```

### UEFI

Add the `efibootmgr` package at the end of the command.

#### UEFI - Arch Linux

Run the `pacstrap` command:

```
pacstrap -K -i /mnt base base-devel linux linux-firmware cryptsetup lvm2 grub networkmanager dhcpcd openssh neovim vim efibootmgr
```

#### UEFI - Artix Linux

Run the `basestrap` command:

```
basestrap -i /mnt base base-devel linux linux-firmware runit elogind-runit cryptsetup lvm2 lvm2-runit grub networkmanager networkmanager-runit neovim vim efibootmgr
```

## Setting up the *chroot* environment

Append the output of the `lsblk -f` command to the `/mnt/etc/default/grub` file:

```
lsblk -f >> /mnt/etc/default/grub
```

If the install has been configured with a swap partition, run the same command, but append the output to `/mnt/etc/fstab`:

```
lsblk -f >> /mnt/etc/fstab
```

### Arch Linux

Use the `genfstab -U /mnt` command to check the output, then add the output to the `/mnt/etc/fstab` file:

```
genfstab -U /mnt >> /mnt/etc/fstab
```

Change into the *chroot* environment:

```
arch-chroot /mnt bash
```

### Artix Linux

Use the `fstabgen -U /mnt` command to check the output, then add the output to the `/mnt/etc/fstab` file:

```
fstabgen -U /mnt >> /mnt/etc/fstab
```

Change into the *chroot* environment:

```
artix-chroot /mnt bash
```

## Making some necessary adjustments

### Time zone

On Arch-based systems, setting the time zone is as easy as using a symlink:

```
ln -s /usr/share/zoneinfo/US/Eastern /etc/localtime
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

### Fix the fstab file

*NB: Because Arch and Artix Linux make use of genfstab/fstabgen (respectively), the only "fixing" that would necessary would be for a **swap partition**.*

Add a line for the swap partition. It should look like the following:

```
UUID=$UUID_swap none swap defaults 0 0
```

The UUID for the swap partition should be found at the bottom of the file, since it was appended earlier with the `lsblk -f >> /mnt/etc/fstab` command. Be sure to remove those lines before saving.

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
127.0.0.1	localhost
::1	    	localhost
127.0.1.1	$HOST $HOST.$localdomain
```

### Enable networking

#### Networking with systems using systemd

On `systemd` systems, which Arch Linux uses by default, the following command will get the `networkmanager` service up and running:

```
systemctl enable systemd-networkd && systemctl enable dhcpcd
```

In addition, if the system is going to receive `ssh` connections, run the following command at this time:

```
systemctl enable sshd
```

#### Networking with systems using runit

On `runit` systems, which is a great init system for Artix Linux, the following command will get the `networkmanager` service up and running:

```
ln -s /etc/runit/sv/NetworkManager /etc/runit/runsvdir/current
```

### Create new main user

First, set a new password for the root user with the following command:

```
passwd
```

Next, create a new user with the `useradd` command. Add the user to the "wheel" group with `-G wheel`, and use `-m` to give that user a home directory.

```
useradd -G wheel -m $USER
```

Once the user has been created, set the user's password with the following command:

```
passwd $USER
```

The last step on Arch-based systems is to confirm that the wheel group can run root-level commands with the `sudo` command. Near the bottom of the "/etc/sudoers" file, uncomment the command that enables users of group "wheel" to run "any command".

```
vim /etc/sudoers
```

NB: For systems using `runit`, it is possible to set the computer up such that booting automatically logs in a user. This is most practical for users who are using FDE. To enable this feature, edit the `/etc/runit/sv/agetty-tty1/conf` file. To this file, look for a line that contains `GETTY_ARGS="--noclear"`, and append the following **AFTER** `--noclear`, but within the quotation marks:

```
--autologin $USER
```

## Kernel initialization

On a system using FDE, it is a requirement to update the `/etc/mkinitcpio.conf` file, so the system knows to load the correct modules in order to unlock the encrypted root partition during bootup.

In the "HOOKS" section, add the following two modules (encrypt and lvm2) between the block and filesystems modules, like so:

```
block encrypt lvm2 filesystems
```

Push the updates to the intial RAM file system with the following command:

```
mkinitcpio -p linux
```

## Bootloader

On a system using FDE, it is a requirement to edit the `/etc/default/grub` using a text editor.

Go to the bottom of the document and clean up the appended lines by commenting them out, removing the superfluous ones, and moving the useful lines up near the top of the file.

Edit the `GRUB_CMDLINE_LINUX_DEFAULT` line to include the following:

```
cryptdevice=UUID=$UUID_sdx2:$LVM_NAME root=UUID=$UUID_/dev/mapper/root
```

As before, `$UUID_sdx2` is the UUID of the block device that was encrypted, and `$LVM_NAME` is the name given to the cryptdevice. `$UUID_/dev/mapper/root` is the UUID of the LVM where the root partition resides. This can vary depending on how the system was set up. Use best judgment and refer to earlier parts of the guide to figure this out. To assist with this, refer to the following bullet point list:

- Users using a swap partition will use the UUID for `/dev/mapper/vgartix-root`.
- Otherwise, users will use the UUID for `/dev/mapper/$LVM_NAME`.

In addition, if there are plans to do virtualization on the system, add to `GRUB_CMDLINE_LINUX_DEFAULT` the following items. For `amd_iommu` and `intel_iommu`, only add the one that corresponds to the CPU of the machine.

```
iommu=pt amd_iommu=on intel_iommu=on
```

For virtualization, if the PCI IDs are known (in relation to GPU passthrough), those can be added to `GRUB_CMDLINE_LINUX_DEFAULT` at this time as well.

```
vfio-pci.ids=abcd:wxyz,...
```

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

After installing GRUB to the system, run the following command:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

---

Congrats on installing an Arch-based Linux system!
