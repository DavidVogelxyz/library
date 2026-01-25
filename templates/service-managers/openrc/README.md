OpenRC related templates
========================

[Back](../README.md)

Default OpenRC service file
---------------------------

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

Whatever `command` is set to (in this case, `/usr/sbin/$RC_SVCNAME`), that file should exist at that path. So, in the example above, `/usr/sbin/newservice` should be an executable script that performs some function.

Once the service file (`/etc/init.d/newservice`) and the script (`/usr/sbin/newservice`) have been created, and have both been made executable, the service can be started with the following command:

```bash
service newservice start
```

To add the service to the default runlevel, run the following command:

```bash
rc-update add newservice
```
