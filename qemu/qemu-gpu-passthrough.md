# PCIe (GPU) Passthrough

## How to set up GPU passthrough

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
sudo nvim /etc/default/grub
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
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Edit the `/etc/mkinitcpio.conf` file:

```
sudo nvim /etc/mkinitcpio.conf
```

Make the following adjustments, where necessary:

`MODULES=(... vfio_pci vfio vfio_iommu_type1 ...)`

`HOOKS=(... modconf ...)`

Update `initramfs`:

```
sudo mkinitcpio -p linux
```

Now, reboot the Linux install, and before starting the VM, add the device to the VMs devices.

```
reboot now
```

Run `lspci -k` to check which drivers control which devices. The GPU (and its subcomponents) to be passed through should have `vfio-pci` as its driver.

```
lspci -k
```

## Extras

### A different way to load `IDs` and `modules`

Edit the following file:

```
sudo nvim /etc/modprobe.d/vfio.conf
```

***I am under the impression that item 1 is unnecessary if IDs are passed through GRUB. This is how it works on my Proxmox server. If true, specifying IDs here is only necessary if they are not included above.***

1. Bind the device IDs (from `lspci -nn`) by adding the line, with IDs separated by ***commas***:

```
options vfio-pci ids=
```

***Line 2 is only necessary if `modules` are not specified in `/etc/mkinitcpio.conf`. I have run QEMU on my desktop without needing it due to having edited `/etc/mkinitcpio.conf`. I also have high confidence that Line 2 without adjustments to `/etc/mkinitcpio.conf` works, and may even be better than adding the modules to `mkinitcpio`.***

2. Enable modules by adding the line:

```
softdep drm pre: vfio-pci
```

"Since this conf file is embedded in the initramfs image, any changes require regenerating a new image each time."

```
sudo mkinitcpio -p linux
```

## The end!

the end


