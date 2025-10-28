Compiling libevent from source, for a local user
================================================

Date written: 2025 July 17, Thursday

How to compile
--------------

Clone the `libevent` repo:

```bash
git clone https://github.com/libevent/libevent ~/.local/src/libevent
```

Change directory into the `libevent` repo:

```bash
cd ~/.local/src/libevent
```

Then, run the `autogen.sh` script:

```bash
sh autogen.sh
```

Once the `autogen.sh` script returns, run the `configure` file:

```bash
./configure --prefix=$HOME/.local --enable-shared
```

Then, run `make` and `make install`:

```bash
make && make install
```

References
----------

- [GitHub - libevent/libevent](https://github.com/libevent/libevent/blob/master/README.md#1-building-and-installation)
