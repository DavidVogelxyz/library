# OpenRC related templates

[Back](../README.md)

## Default OpenRC service file

🚨 **Remember to set the file as executable before starting the service!** 🚨

This file should exist within `/etc/init.d`.

`$RC_SVCNAME` and `$SVCNAME` are set by the name of the service file. Therefore, a file found at `/etc/init.d/newservice` will have both variables set to `newservice`.

```bash
#!/sbin/openrc-run

name="busybox $RC_SVCNAME"
command="/usr/sbin/$RC_SVCNAME"
#command_args="arg1"
pidfile="/run/$SVCNAME.pid"
command_background="yes"

depend() {
    need localmount
}
```
