# Tips for creating optimized Windows VMs on QEMU

## General rules

When booting QEMU for the first time, make sure that `virt-manager` is connected to the QEMU hypervisor. Right click and connect if needed.

Also, change preferences to `allow XML editing`, and confirm via `Connection Details` that the virtual network is `active`.

## Windows 10 (2015) & Windows 11 (2021)

When creating a new VM in QEMU, be sure to check "customize configuration before install." Everything else can be clicked through, as there will be a chance to make changes during the "customization" step.

During the "customization before install" stage, make the following changes:

### Edit XML

Select the "overview" section, and then select the "XML" tab.

Delete the two lines in the clock section containing these segments:

```
timer name="rtc"
timer name="pit"
```

Change the line with the following segment to `yes`.

```
timer name="hpet"
```

### CPU Topology

Switch back to the GUI from the "XML" tab and select the "CPUs" section. Check the box to "manually set CPU topology".

It is prefererable that the host machine be able to do hyper-threading.

If SMT is enabled, then configure based on CPU topology. For example, a 8 core & 16 thread CPU should have 2 threads for every CPU.

If SMT is **not** enabled, then use "cores" as a substitute for "total threads", and keep "threads" at '1'.

Using the example of the 8 core & 16 thread CPU, and provising half of the total CPU resources:
- SMT enabled: 1 socket, 4 cores, 2 threads
- SMT disabled: 1 socket, 8 cores, 1 thread

### Storage

Select the "SATA Disk 1", which is the current storage drive for the VM, and change the type to "VirtIO".

It should now show as VirtIO Disk 1.

### Network

Select the NIC section, and change the type to a VirtIO NIC.

### Drivers

In order to load the VirtIO drivers correctly, a second CD drive is required.

Select "Add Hardware" at the bottom, and add a storage drive. Change the type to "CD drive," and mount the correct ISO image.

### Specific to Windows 10 (2015)

If installing Windows 11, continue to the next step. However, at this point, a Windows 10 VM should be ready for install, and the guide will continue at [post-installation optimization](#post-installation-optimization).

### Specific to Windows 11 (2021)

Everything that was included in the "Windows 10 & 11" applies to a Windows 11 VM; however, there are additional considerations when setting up a Windows 11 VM.

#### TPM (Trusted Platform Modules)

The most significant consideration is the requirement that any computer running Windows 11 be equipped with a TPM, a hardware security module (HSM). This also applies to a VM. Windows 10 allowed for, but did not require, TPMs.

While the option exists to passthrough a TPM that resides on the host computer's motherboard, it is far simpler to emulate a TPM using the `swtpm` package (named as such in both Debian and Arch package repositories).

Then, select "Add Hardware" and add an emulated TPM.

#### First boot - BIOS shell

Due to the way that the clock settings were edited, the first boot will dump the user into a shell. Type `exit` to go to a more standard BIOS screen, and select the ISO boot option.

#### "Internet required to set up Microsoft account"

In Windows 10, it was far easier to expose the option "I have no internet; make a local account". Options included "having no ethernet plugged in" and "selecting the option."

In Windows 11, Microsoft will attempt to shame the user for not having internet. If a wireless adapter exists, it will require the user to connect.

However, using the "shift-F10" keymap will expose a shell. Running `oobe/BypassNRO` will restart the VM and allow for the user to see the "make a local account" option.

#### User setup

There is a small tip for user creation: do ***not*** enter a password for the local user.

Foregoing a password at this step will save the user from having to select 3 security questions and entering responses.

### Post-installation optimization

A user installing Windows 10 or Windows 11 should arrive at this step with a fully installed Windows VM.

#### Install the remaining VirtIO drivers

Right click on the "start menu icon" (which should be the Windows logo) and select "Device Manager."

Within the Device Manager, there will be numerous different devices that have the ⚠️ present over their icon. Right-click these devices and select "update drivers." Then, select the mounted VirtIO iso ***and subdirectories*** as the source of the drivers. Device Manager will then find the correct driver, and it can be installed.

Do this for every device that needs it. Drivers that should be seen receiving updates include: network driver, display driver, and "balloon" driver. When all drivers have been updated, restart the VM.

Upon successful reboot, the VM should be able to access the internet. Connect to wireless if necessary.

#### WinUtil (from Chris Titus Tech)

Most of the spyware and telemetry that Microsoft bundles into their Windows OS can be neutralized and removed by using [WinUtil](https://github.com/ChrisTitusTech/winutil), created by Chris Titus Tech.

To do this, right click on the "start menu icon" (which should be the Windows logo) and select "PowerShell (Admin)". Enter the following into the command line, and wait a few moments for the software to run:

`irm https://christitus.com/win | iex`

When it starts, navigate over to the "tweaks" section, and select either "desktop" or "laptop". Then, at bottom right, select "run tweaks." If it doesn't seem like anything is happening, bring the PowerShell window back and it should show that commands are being executed. When the utility has completed running the tweaks, a small pop-up will appear (that may only be true if the utility is the primary window upon completion).

#### GPU passthrough

***NB: These tips are primarily focused on what needs to happen within the Windows VM in order to get the GPU to pass-through optimally. For tips related to setting up passthrough in QEMU, see other documentation.***

First, the hardware needs to be added to the VM. Using virt-manager, edit the VM's configuration and add the hardware correctly.

Be sure to include all components of the GPU. In other words, a GPU may have more than one IOMMU item. For example, it's possible to have four components: one video, and three audio. Make sure that all are added.

With the hardware added, power on the VM, and use Windows and a web browser to get GPU drivers installed. For example, with an Nvidia GPU, download the drivers (or the GeForce Experience program) and install the drivers. This can be confirmed after a reboot, as Device Manager (and other Windows utilities) will be able to see the GPU.

At this stage, the GPU should be outputting video through its display outputs. However, it is likely just a blank desktop with no cursor. To force the GPU as the main and sole display on future reboots, open Device Manager, navigate to "Display/Red Hat QXL" and "disable the driver." This will revert the non-GPU video output to 1280x800. Simply restart the VM and begin using the alternate monitor setup.

#### Mouse troubles

On some occasions when creating Windows VMs, I have noticed an issue where the mouse cursor seems to stick to the left and right sides of the screen. When this happens, the cursor cannot move up or down until it retreats from the edge of the screen. This does not happen with the top or bottom "horizontal" borders, and only the left and right "vertical" borders. As bad as this was on the desktop, it was worse in video games, as the issue persisted without the user knowing where the cursor was.

The fix was relatively simple, though easily overlooked. In the hardware configuration for the VM, there will be an item under "Mouse", named "Tablet". Remove this "Tablet" item, and the mouse will work as intended.

## Windows XP (2001)

## Windows 7 (2009)

## Windows Vista (2007)
