Compiling ncurses from source, for a local user
===============================================

Date written: 2025 June 16, Monday

How to compile
--------------

Clone the `ncurses` repo:

```bash
git clone https://github.com/mirror/ncurses ~/.local/src/ncurses
```

Change directory into the `ncurses` repo:

```bash
cd ~/.local/src/ncurses
```

Configure to install for the user, instead of system-wide:

```bash
./configure --prefix=$HOME/.local
```

Compile `ncurses`:

```bash
make
```

Install `ncurses` for the user:

```bash
make install
```
