Compiling cmake from source, for a local user
=============================================

Date written: 2025 June 16, Monday

How to compile
--------------

Clone the `cmake` repo:

```bash
git clone https://github.com/Kitware/CMake ~/.local/src/cmake
```

Change directory into the `cmake` repo:

```bash
cd ~/.local/src/cmake
```

Configure to install for the user, instead of system-wide:

```bash
./configure --prefix=$HOME/.local
```

Another way to configure:

```bash
./bootstrap
```

Compile `cmake`:

```bash
make
```

Install `cmake` for the user:

```bash
make install
```
