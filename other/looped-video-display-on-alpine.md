# Looping a video file on a display, using a mini-PC running Alpine Linux

NB:

- This guide has been tested using a mini-PC running Alpine Linux.
- On multiple occasions, this guide makes reference to the `service` command. Know that `service` functions the same way as `rc-service`.
- In addition, `rc-update add $SERVICE` is a shorter way to write `rc-update add $SERVICE default`.
    - Both commands will add the `$SERVICE` to the "default" run level.
- When using a tty on Alpine Linux, "Ctrl-Alt-Del" reboots the computer.

## Table of contents

- [Making changes to the BIOS of the mini-PC](#making-changes-to-the-bios-of-the-mini-pc)
- [Installing Alpine Linux](#installing-alpine-linux)
- [Initial configuration](#initial-configuration)
- [Configuring the video loop](#configuring-the-video-loop)
- [References](#references)

## Making changes to the BIOS of the mini-PC

In order to get the mini-PC to work properly as a full-time TV display, some changes need to be made to BIOS settings.

Those changes include:

- Disable "trusted boot"
    - This will allow the mini-PC to boot the USB drive containing Alpine Linux.
- Disable "suspend"
    - This will enable the mini-PC to run the looped video, so long as it has power.
- Disable "TPM" (Trusted Platform Module)
    - This is another setting that needs to be disabled in order for the mini-PC to boot Alpine Linux.
- Enable "automatically boot when power is supplied"
    - This will allow the mini-PC to restart if there is some sort of disruption to the power supply.
- Set the OS to "Intel Linux"
    - This will allow the mini-PC to boot the USB drive containing Alpine Linux.

Once these changes are made, save and exit the BIOS. Reboot the mini-PC with the USB containing the Alpine Linux ISO connected to the mini-PC, and enter the BIOS settings once more.

Now, boot from the USB, and enter the Alpine Linux live USB environment.

## Installing Alpine Linux

This step should not require much explanation. Install Alpine Linux onto the mini-PC using `setup-alpine` from within the live USB environment. Consult the [Alpine Linux documentation](https://docs.alpinelinux.org/user-handbook/0.1a/Installing/setup_alpine.html) regarding `setup-alpine` if instruction is needed.

## Initial configuration

Follow this guide on [configuring a server running Alpine Linux](/servers/configuring-alpine-server.md) to set up a new user account and secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

## Configuring the video loop

With `ssh` secured, the next step is to install some packages required to make the video loop and display properly.

Update the package repositories, and install the following packages:

```
apk update && apk add ffmpeg intel-media-driver mesa-dri-gallium mesa-va-gallium mpv
```

Then, open "/etc/init.d/videoloop" in a text editor:

```
vim /etc/init.d/videoloop
```

Add the following to the new file:

```
#!/sbin/openrc-run

name="busybox $RC_SVCNAME"
command="/usr/sbin/$RC_SVCNAME"
command_args="/usr/share/videoloop/video.mp4"
pidfile="/run/$SVCNAME.pid"
command_background="yes"

depend() {
        need localmount
}
```

Make the OpenRC service file executable:

```
chmod +x /etc/init.d/videoloop
```

Open "/usr/sbin/videoloop" in a text editor:

```
vim /usr/sbin/videoloop
```

Add the following to the new file:

```
#!/bin/sh

# for displays that are a standard 16:9 widescreen
mpv $* -vf crop=1920:1080,setdar=16/9,scale=1920:1080 -loop

# for displays that are a standard 16:9 widescreen, but are rotated sideways
#mpv $* -vf crop=1920:1080,transpose=2,setdar=16/9,scale=1920:1080 -loop
```

Make the new file executable:

```
chmod +x /usr/sbin/videoloop
```

Now, create the "/usr/share/videoloop" directory:

```
mkdir -pv /usr/share/videoloop
```

Add the video file to the "/usr/share/videoloop" directory, and test the `videoloop` command with the following:

```
videoloop $PATH_TO_FILE
```

Once the command is confirmed to work, start the `videoloop` service with the following:

```
service videoloop start
```

Also, remember to add the service to the default runlevel, so the video will loop on startup:

```
rc-update add videoloop
```

## References

- [Alpine Linux documentation - setup-alpine](https://docs.alpinelinux.org/user-handbook/0.1a/Installing/setup_alpine.html)
