Compiling tmux from source, for a local user
============================================

Date written: 2025 July 17, Thursday

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

How to compile
--------------

Clone the `tmux` repo:

```bash
git clone https://github.com/tmux/tmux ~/.local/src/tmux
```

Change directory into the `tmux` repo:

```bash
cd ~/.local/src/tmux
```

Checkout the version that should be installed:

```bash
git checkout stable
```

Then, run the `autogen.sh` script:

```bash
sh autogen.sh
```

Once the `autogen.sh` script returns, run the `configure` file:

```bash
PKG_CONFIG_PATH=$HOME/.local/lib/pkgconfig ./configure --prefix=$HOME/.local
```

Then, run `make` and `make install`:

```bash
make && make install
```

`tmux` should now be installed to `~/.local/bin/tmux`!

Make sure to create an alias to use the locally installed version. In addition, use the following command to direct `tmux` to use the local libraries:

```bash
LD_LIBRARY_PATH=$HOME/.local/lib $HOME/.local/bin/tmux
```

Troubleshooting
---------------

If compiling fails because the system is missing `libevent`, it can be installed locally through the package manager.

If the package manager is unavailable, please refer to [compiling libevent from source](compile-libevent.md) for more information on compiling `libevent`.

Now that `libevent` should be installed for the local user, proceed with building `tmux` from source by running `make && make install`.

References
----------

- [GitHub - tmux/tmux](https://github.com/tmux/tmux?tab=readme-ov-file#from-version-control)
- [GitHub - tmux/tmux - Building dependencies](https://github.com/tmux/tmux/wiki/Installing#building-dependencies)
