Tips for PCIe (GPU) passthrough on Proxmox
==========================================

Part of a guide on setting up a [Proxmox server](install-proxmox.md).

How to set up GPU passthrough
-----------------------------

### BIOS side

Intel users look for `Intel VT-d`.

AMD users look for `AMD-Vi`.

Some motherboards don't have options for either; instead, it will be listed as `IOMMU`.

In either case, IOMMU needs to be enabled.

### Operating System side

Gather the necessary information using `lspci`:

With `lspci`, and switch `-nn`, grab the PCI IDs for the video card and all of its extra components (like the audio!):

```
lspci -nn

lspci -k

lspci -k -s 01:00
```

Then, input that information into the correct spots:

Edit `/etc/default/grub`:

```
nvim /etc/default/grub
```

- Enable IOMMU.
  - For AMD, add the switch `amd_iommu=on`.
    - I hear this is unnecessary for **AMD only.**
  - For Intel, add the switch `intel_iommu=on`.
- Add the switch `IOMMU=pt`.
  - "This will prevent Linux from touching devices which cannot be passed through."
- Bind the device IDs (from `lspci -nn`).
  - `vfio-pci.ids=`
    - Comma separated list

Regenerate the bootloader image.

```
update-grub
```

Edit the `/etc/modules` file:

```
nvim /etc/modules
```

Add the following modules:

```
vfio
vfio_pci
vfio_iommu_type1
```

Update `initramfs`:

```
update-initramfs -u -k all
```

Now, reboot the Proxmox install, and before starting the VM, add the device to the VMs devices.

```
reboot now
```

Run `lspci -k` to check which drivers control which devices. The GPU (and its subcomponents) to be passed through should have `vfio-pci` as its driver.

```
lspci -k
```

Extras
------

### A different way to load `IDs` and `modules`

Edit the following file:

```
sudo nvim /etc/modprobe.d/vfio.conf
```

***Item 1 is CONFIRMED unnecessary if IDs are passed through GRUB. Specifying IDs here is only necessary if they are not included above.***

1. Bind the device IDs (from `lspci -nn`) by adding the line, with IDs separated by ***commas***:

```
options vfio-pci ids=
```

***Line 2 is only necessary if `modules` are not specified in `/etc/modules`. However, I have not tried using Line 2 without the additions to `/etc/modules`.***

2. Enable modules by adding the line:

```
softdep drm pre: vfio-pci
```

"Since this conf file is embedded in the initramfs image, any changes require regenerating a new image each time."

```
update-initramfs -u -k all
```

References
----------

- [YouTube - Techno Tim - Before I do anything on Proxmox, I do this first...](https://www.youtube.com/watch?v=GoZaMgEgrHw)
    - Reference for GPU passthrough on Proxmox
- [Proxmox documentation - PCI Passthrough](https://pve.proxmox.com/wiki/PCI_Passthrough)
