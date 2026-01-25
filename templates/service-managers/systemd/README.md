Systemd related templates
=========================

[Back](../README.md)

Default `systemd` service file
------------------------------

New service files should exist within `/etc/systemd/system`.

```
[Unit]
Description=<NAME>
After=network.target

[Service]
ExecStart=<PATH>
User=<USER>
Group=<GROUP>
Type=simple
Restart=on-failure
TimeoutSec=300
RestartSec=30
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
```

To explain:
- `<NAME>` should be a name or description of the service:
    - Example:
        - `Description=My new service`
- `<PATH>` should be the path to the executable:
    - Example:
        - `ExecStart=/usr/local/bin/program`
- `<USER>` should be the user that `systemd` "becomes" to run the executable:
    - Example:
        - `User=test`
    - Note:
        - This is important to set -- otherwise, the service will run as `root`.
- `<GROUP>` should be the group that `systemd` "belongs to" when running the executable:
    - Example:
        - `Group=test`
    - Note:
        - As with `User`, it's important to set this -- otherwise, the service will belong to the `root` group.
- `PrivateTmp` provides a private `/tmp` and `/var/tmp` for the process.
- `ProtectSystem` mounts `/usr`, `/boot` and `/etc` as "read-only" for the process.
- `NoNewPrivileges` disallows the process, and its child processes, from gaining new privileges.
- `PrivateDevices` uses a new `/dev` namespace for the process:
    - That new `/dev` namespace is only populated with API pseudo devices, such as:
        - `/dev/null`
        - `/dev/zero`
        - `/dev/random`
- `MemoryDenyWriteExecute` denies the creation of writable and executable memory mappings:
    - Note:
        - Some processes will require `MemoryDenyWriteExecute` to be enabled:
            - If this is the case, it is not enough to comment out, or remove, this entry.
            - Instead, `MemoryDenyWriteExecute` must be set to `true`.
