# Tips for setting up a Windows VM in Proxmox

The one tip that's worth mentioning is a small hiccup that can occur when installing Windows and using the VirtIO drivers.

At the point of setting up the storage drive for the Windows install, the installer asks about loading drivers. Many times, there's an urge to hit the "browse" button to navigate over to the drive that contains the VirtIO ISO. This seems to cause issues. Instead, just hit the "OK" button.

Later on, when the install is complete and it's time to install the remaining VirtIO drivers using "Device Manager", it is appropriate to find the drivers by browsing over to the drive with the VirtIO ISO and then "searching recursively" within that directory.
