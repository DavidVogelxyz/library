How to compile `ueberzugpp` on Rocky Linux
==========================================

Date added: 2025 December 27, Saturday

🚨 **THIS GUIDE IS INCOMPLETE!**
- currently stopped on `libsixel`:
    - while `libsixel` was successfully built from source, it appears it may have caused issues with text rendering:
        - after reboot, was able to confirm that all text rendered in dwm is now rendered as symbols 😬

Notes
-----

To compile successfully, first install the "Development Tools" group:

```bash
dnf group install "Development Tools"
```

To install `vips` and `vips-devel`, first run the following command to enable the "PowerTools" repo:

```bash
dnf config-manager --enable crb
```

Then, run the following command to enable the "Remi" repos:

```bash
dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

Dependencies:

```
opencv
opencv-devel
tbb
tbb-devel
vips
vips-devel
```

Clone the repo:

```bash
git clone https://github.com/jstkdng/ueberzugpp
```

Change directory into the repo. Then, run the following commands:

```bash
mkdir -pv build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
```

### libsixel

To compile `libsixel` from source, clone the repo:

```bash
git clone https://github.com/saitoha/libsixel
```

Change directory into the repo. Then, run the following commands:

```bash
./configure
make
sudo make install
```
