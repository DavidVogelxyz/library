# Extending Windows VM Disk Image

## Table of contents

- [Introduction](#introduction)
- [How-to](#how-to)
- [Clean up](#clean-up)
- [References](#references)

## Introduction

This guide will assist the user in extending a Windows partition.

Normally, this can easily be done using the "Disk Management" utility. However, if the "Windows recovery partition" is "in the way", then it will not be as easy to extend the partition with the root file system.

## How-to

Start with `diskpart`:

```
diskpart
```

Once in `diskpart`, run these commands:

```
DISKPART> list disk
DISKPART> select disk <the-disk-number-where-current-recovery-partition-is-located>
DISKPART> list partition
DISKPART> select partition <the-disk-number-of-current-recovery-partition>
DISKPART> assign letter=O
```

Then, run these commands:

```
dism /Capture-Image /ImageFile:C:\recovery-partition.wim /CaptureDir:O:\ /Name:"Recovery"
dism /Apply-Image /ImageFile:C:\recovery-partition.wim /Index:1 /ApplyDir:N:\
reagentc /disable
reagentc /setreimage /path N:\Recovery\WindowsRE
reagentc /enable
```

## Clean up

To get rid of the leftover stuff, run `diskpart` again:

```
diskpart
```

Once in `diskpart`, run these commands (UEFI-specific):

```
DISKPART> select volume N
DISKPART> set id="de94bba4-06d1-4d40-a16a-bfd50179d6ac"
DISKPART> gpt attributes=0x8000000000000001
DISKPART> remove
```

To delete the recovery partition, run the following:

```
DISKPART> select volume O
DISKPART> delete partition override
```

## References

- [SuperUser - How to expand the Windows partition when the Recovery one is in the way?](https://superuser.com/questions/1354574/how-to-expand-the-windows-partition-when-the-recovery-one-is-in-the-way/1825572#1825572)
    - Somewhat helpful
- [SuperUser - How to move the recovery partition on Windows 10?](https://superuser.com/questions/1453790/how-to-move-the-recovery-partition-on-windows-10/1596291#1596291)
    - **Reference that was the basis for this guide**
