How to compile `lf` on Rocky Linux
==================================

Date added: 2025 December 27, Saturday

Notes
-----

Dependencies:

```
golang
```

After installing `golang`, run the following command:

```bash
env CGO_ENABLED=0 go install -ldflags="-s -w" github.com/gokcehan/lf@latest
```

Once the command completes, the binary can be found at `~/.local/share/go/bin/lf`.

To run `lf`, do one of the following:
- configure `lfub` to execute `~/.local/share/go/bin/lf`
- create a symlink in `~/.local/bin` that links to `~/.local/share/go/bin/lf`
