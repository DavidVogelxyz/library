How to compile `xwallpaper` on Rocky Linux
==========================================

Date added: 2025 December 25, Thursday

Notes
-----

To install `xcb-util-devel`, first run the following command to enable the "PowerTools" repo:

```bash
dnf config-manager --enable crb
```

Dependencies:

```
autoconf
automake
libjpeg-turbo-devel
pixman-devel
xcb-util-devel
xcb-util-image-devel
```

Clone the repo:

```bash
git clone https://github.com/stoeckmann/xwallpaper
```

Change directory into the repo. Then, run the following commands:

```bash
./autogen.sh
./configure
make
sudo make install
```
