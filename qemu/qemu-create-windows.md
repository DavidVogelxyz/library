# Creating Optimized Windows VMs on QEMU

## General rules

On the first-ever boot, make sure that `virt-manager` is connected to the QEMU hypervisor. Right click and connect if needed.

Also, change preferences to `allow XML editing`.

Confirm via `Connection Details` that the virtual network is `active`.

## Windows 10 (2015) / Windows 11 (2021)

Customize configuration before install.

### Edit XML

Delete two lines under clock section:

`timer name="rtc"`

`timer name="pit"`

Change the following lines to `yes`.

***EDIT: Since creating my Windows VMs, I have had to go in and switch this to `off`. No longer suggested to turn on.***:

***DOUBLE EDIT: Since writing this guide, I checked again. Seems it needs to be on. Will look into further.***

`timer name="hpet"`

### CPU Topology

Use GUI for this.

On my machine, I ***only*** adjust `cores`. I ***do not ever*** adjust sockets and threads; both always stay at `1`.

Example: I currently run a 12 core, 24 thread CPU. In order to give the VM half of my computer's power, I use the number `12`, as if it was threads, but use the unit `cores`. I keep the other two values at `1`.

I have seen suggestions to the contrary, that I should try setting it at `6 cores`, but try the threads at `2 threads`. However, I swear I tried that and it didn't work.

### Other settings

Go to SATA Disk 1 and make it VirtIO Disk 1.

Go to NIC and make it a VirtIO NIC.

### Drivers

Remember to use the VirtIO drivers package in Windows.

Will need to add a second CD-ROM drive to mount the drivers ISO while setting up the Windows guest.

## Windows XP (2001)

## Windows 7 ()

## Windows Vista ()
