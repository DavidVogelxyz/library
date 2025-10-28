Compiling vim from source, for a local user
===========================================

Date written: 2025 June 16, Monday

How to compile
--------------

Clone the `vim` repo:

```bash
git clone https://github.com/vim/vim ~/.local/src/vim-src
```

Change directory:

```bash
cd ~/.local/src/vim-src
```

Checkout the correct version:

```bash
git checkout stable
```

Configure to install for the user, instead of system-wide:

```bash
./configure --prefix=$HOME/.local
```

Compile `vim`:

```bash
make
```

Install `vim` for the user:

```bash
make install
```

Troubleshooting
---------------

If compiling fails because the system is missing `libncurses-dev`, it can be installed locally through the package manager.

If the package manager is unavailable, please refer to [compiling ncurses from source](compile-ncurses.md) for more information on compiling `ncurses`.

Once `ncurses` is installed, return to the `vim` repo and run:

```bash
LDFLAGS=-L$HOME/.local/src/ncurses/lib ./configure --prefix=$HOME/.local --config-cache --with-tlib=ncurses
```

Then, proceed with `make` and `make install`.

References
----------

- [GitHub - vim/vim - cross compile vim with tlib](https://github.com/vim/vim/issues/2058)
