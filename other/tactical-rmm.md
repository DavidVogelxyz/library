# Tactical RMM

## Table of contents

- [Installation](#Installation)
- [Troubleshooting a local deployment](#Troubleshooting-a-local-deployment)
- [Step one - tmux](#Step-one---tmux)
- [Step two - self-signed SSL certificates](#Step-two---self-signed-SSL-certificates)
- [Step three - locales](#Step-three---locales)
- [Step four - domains & DNS](#Step-four---domains-&-DNS)
- [Step five - connect to the dashboard](#Step-five---connect-to-the-dashboard)
- [Step six - removing the insecure flag](#Step-six---removing-the-insecure-flag)
- [Step seven - adding SSL certificates signed by a root CA](#Step-seven---adding-SSL-certificates-signed-by-a-root-CA)

## Installation

Use [the guide](https://docs.tacticalrmm.com/install_server/) provided by the Tactical RMM team. However, be sure to read the remainder of this guide ***before*** attempting the install, as it will prevent what seem to be "common errors."

## Troubleshooting a local deployment

There are handful of things that should be done in order to have everything work correctly in a local deployment.

### Step one - tmux

Install `tmux` and run the "tactical" user's command in the `tmux` shell. That way, if there are any issues with connection via `ssh`, the connection isn't dropped. If `tmux` is used, it will be possible to reattach to the install script and continue. Without `tmux`, it's possible to drop out of the install with no way back in. The only solution at this point is to reinitialize the VM and start over.

### Step two - self-signed SSL certificates

When doing a local deployment that is ***not*** a production deployment, make sure to run the install script with the "--insecure" flag.

```
./install.sh --insecure
```

This will generate self-signed SSL certificates. Without this, the installation will fail. The only solution at this point is to reinitialize the VM and start over.

### Step three - locales

At the beginning of the install, the script may shout about `locales` not being set up properly. This is an irritating issue. The install script *specifically* wants the user to run the following:

```
sudo dpkg-reconfigure locales
```

Then, confirm that *only* "en_US.UTF-8" is selected. On the next screen, generate the locales using "en_US.UTF-8".

One may believe that editing the "/etc/locale.gen" file and then running `locale-gen` would result in the same outcome. For whatever reason, it does not.

### Step four - domains & DNS

At the beginning of the install (after the potential "locales" issue), the script will ask the user to input domains for the different services. Use the following schema:

```
api         = api.$DOMAIN
rmm         = rmm.$DOMAIN
mesh        = mesh.$DOMAIN
domain      = $DOMAIN
user e-mail = $EMAIL
```

"$DOMAIN" may look like the following:

```
$HOSTNAME.$FQDN.$TLD
```

An example of this may be the following: `tactical-rmm.example.com`

Therefore, the API domain would be: `api.tactical-rmm.example.com`

This is fine, but will likely require additional configuration.

- One level of configuration will be at the level of the router. It will require a DNS entry for that server and the "tactical-rmm.example.com" domain.
- Another level of configuration will be the "hosts" file. Every computer that will access the server (including clients; also known as "agents") will need an entry for the IP of the server, and all of the different hostnames and domains. An example for a Linux machine ("/etc/hosts") follows:

```
$IP_ADDRESS     tactical-rmm tactical-rmm.example.com api.tactical-rmm.example.com rmm.tactical-rmm.example.com mesh.tactical-rmm.example.com
```

Windows users would add the same entry into their "hosts" file.

At this point, the install should run correctly and without error. Create a user and password for the admin when requested, and collect the TOTP information as well as the Mesh Central information. Save this information.

### Step five - connect to the dashboard

To use the example from before, https://tactical-rmm.example.com will bring up the dashboard login. However, using this link *will* result in a "backend is offline" error message.

Therefore, be sure to access the dashboard from https://rmm.tactical-rmm.example.com URL.

This should work, and will direct the user to input a TOTP code.

### Step six - removing the insecure flag

Because of the "--insecure" flag that was set during install, there are many functions that Tactical RMM will not allow the user to perform. This includes creating executables to allow clients (known as "agents") to be added to the dashboard for remote management.

In order to resolve this, a simple trick can be employed.

Use vim, or a preferred text editor, to open up the following file:

```
vim /rmm/api/tacticalrmm/tacticalrmm/local_settings.py
```

In this file, look for the following entry, and delete it:

```
TRMM_INSECURE = True
```

Restart the server. Tactical RMM will no longer throw errors about attempting to do tasks while using "insecure" self-signed SSL certificates.

### Step seven - adding SSL certificates signed by a root CA

In conjunction with step six, an additional measure can be taken to secure the server with SSL certificates that have been signed by a root CA (certificate authority).

Follow the guide in this Git repo about creating and setting up certificates.

The one key detail to note is that Tactical RMM will require a wildcard certificate. However, creating a wildcard certificate for `*.example.com` will not work, as the wildcard only seems to work on a single domain level above the listed domain.

Therefore, a wildcard certificate would need to be created for `*.tactical-rmm.example.com`. Certificates made with this as the listed domain will be approved by the root CA.

Simply replace the "cert.pem" (public key) and "key.pem" (private key) with those found in the "/etc/ssl/tactical" directory. Then, restart the server. So long as the root CA has been added to the machine, either at the browser or the OS level, the certificate will register as valid.

Congrats on getting the Tactical RMM server deployed!
