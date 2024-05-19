# Resizing volumes using LVM (Logical Volume Management)

## Introduction

LVM is a great tool for creating resizable partitions in Linux. LVM allows a user to increase the storage capacity of a drive, and then extend the volumes to incorporate the additional storage space. However, doing this can be a bit confusing without a walkthrough.

To explain a bit further, take the example of an Ubuntu install on a 200GB SSD where LVM is used. In addition, this computer currently has 8GB of RAM. On the device 'sda', two partitions are created: "sda1" (boot) and "sda2" (root). Within "sda2", a physical volume (PV) is created that encompasses the entirety of "sda2". That PV is added to a volume group (VG) labeled "ubuntu-vg" -- therefore, the highest level abstraction is the VG "ubuntu-vg", with the PV contained within it. Then, that PV is divided up into two logical volumes (LVs): one to be used as a swap device, and the other to contain the root file system.

Now, imagine upgrading the SSD from a 200GB storage drive to a 1TB drive, and the RAM from 8GB to 32GB. After imaging the original drive using the guide found in the [Library repo](https://github.com/davidvogelxyz/library) and deploying the installation image onto the new storage drive, the file system would still only register a capacity of 200GB. Adjustments would need to be made to increase the storage space from 200GB to 1TB. At the same time, the swap device would also need to be increased, from 8GB to 32GB.

This is where this guide comes in.

## Table of contents

There are multiple steps to resizing all volumes correctly. They are as follows:

- [Resize the partition (volume group)](#resize-the-volume-group)
- [Resize the physical volume](#resize-the-physical-volume)
- [Resize the logical volume](#resize-the-logical-volume)
- [Perform a 'file system check'](#perform-a-'file-system-check')
- [Resize the file system](#resize-the-file-system)

## Resize the volume group

Resize the partition (volume group) using a live USB environment, as the partition being resized up ***must be*** unmounted.

This can most easily be done by using a GUI tool such as `gparted`. In `gparted`, resizing the parition is as simple as selecting the partition ("sda2" in the above example) and dragging the slider to fill all the free space that now exists on the new drive.

Use the following command before and after resizing the volume group (partition) to confirm the change:

```
vgdisplay
```

## Resize the physical volume

Once the partition (volume group) has been resized, leave the live USB environment and boot back into the machine, using the new 1TB storage drive.

Resize the physical volume using the `pvresize` command, and use `pvdisplay` before and after to see the changes. When using `pvresize`, the only argument it needs is the path for the volume group ("ubuntu-vg" in the above example). This is because the PV is already a constituent of the VG; therefore, `pvresize` will attempt to take the PV and resize it to the maximum capacity of the VG.

```
pvdisplay
```

```
pvresize /dev/$ROOT_PHYSICAL_VOLUME
```

```
pvdisplay
```

## Resize the logical volume

With the VG and the PV resized, it is now possible to increase the sizes of the logical volumes (LVs).

By using `lvdisplay` before and after the adjustments, it is possible to see clearly what changes were made. First, resize the swap device with `lvresize -L +24GB` to add 24GB to it, increasing the capacity from the original 8GB up to 32GB. Then, resize the root file system's volume using `lvresize -l +100%FREE`, which will give the LV all the remaining space available.

```
lvdisplay
```

```
lvresize -L +24G /dev/ubuntu-vg/swap_1
```

```
lvresize -l +100%FREE /dev/ubuntu-vg/root
```

```
lvdisplay
```

## Perform a 'file system check'

While the VG, PV, and LVs have all been resized, it is possible to see with the following command that the file system still hasn't been extended to encompass all the new space.

```
df -Th
```

First, perform a file system check with the following command:

```
e2fsck -f /dev/ubuntu-vg/root
```

## Resize the file system

Then, resize the file system for the root file system using the following command:

```
resize2fs /dev/ubuntu-vg/root
```

Confirm the changes with:

```
df -Th
```

Congrats on resizing storage volumes using LVM!
