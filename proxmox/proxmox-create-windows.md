# Tips for setting up a Windows VM in Proxmox

The one tip that's worth mentioning is a small hiccup that can occur when installing Windows and using the VirtIO drivers.

At the point of setting up the storage drive for the Windows install, the installer asks about loading drivers. Many times, there's an urge to hit the "browse" button to navigate over to the drive that contains the VirtIO ISO. This seems to cause issues. Instead, just hit the "OK" button.

After the install completes, install the remaining VirtIO drivers using the "Device Manager" utility. Once "Device Manager" is running, right click on a device that requires drivers, and search the VirtIO drive recursively to find the appropriate drivers. There should be about 3-4 drivers to install, including the "balloon" driver, the "serial" driver, and the "graphics" driver.
