Compiling neovim from source
============================

Date written: 2025 June 16, Monday

Before compiling
----------------

First, check if `cmake` is available:

```bash
type -a cmake
```

If `cmake` is not available, there are a few options:
- Install `cmake` as a package from the system's package manager
- If `cmake` cannot be installed, it may be available as a module
- If `cmake` cannot be installed, nor is it available as a module, [compile cmake from source](compile-cmake.md)

If `cmake` is not installed/installable, but the environment is an HPC system, `cmake` may be available as a module.

To check if `cmake` is available as a module, run:

```bash
module av cmake
```

If `cmake` is available as a module, but not available with `type -a`, load the module:

```bash
module load cmake
```

If `cmake` isn't installed, cannot be installed, and isn't available as a module, then it's possible to [compile cmake from source](compile-cmake.md).

Once `cmake` is confirmed to be available on the system, continue.

Preparing neovim
----------------

Clone the `neovim` repo:

```bash
git clone https://github.com/neovim/neovim ~/.local/src/neovim
```

Change directory into the repo:

```bash
cd ~/.local/src/neovim
```

Checkout the version that should be installed:

```bash
git checkout stable
```

Compiling neovim system-wide
----------------------------

Compile `neovim` for the system:

```bash
make CMAKE_BUILD_TYPE=RelWithDebInfo
```

Install `neovim` system-wide:

```bash
sudo make install
```

Compiling neovim for a local user
---------------------------------

Compile `neovim` for the user, instead of system-wide:

```bash
make CMAKE_BUILD_TYPE=RelWithDebInfo CMAKE_INSTALL_PREFIX=${HOME}/.local
```

Install `neovim` for the user:

```bash
make install
```
