# Tactical RMM server, running on Debian

## Table of contents

- [Initial configuration](#initial-configuration)
- [Installing Tactical RMM](#installing-tactical-rmm)
- [Troubleshooting a local deployment](#troubleshooting-a-local-deployment)
    - [tmux](#tmux)
    - [Self-signed SSL certificates](#self-signed-ssl-certificates)
    - [locales](#locales)
    - [Domains and DNS](#domains-and-dns)
    - [Connect to the dashboard](#connect-to-the-dashboard)
    - [Removing the insecure flag](#removing-the-insecure-flag)
    - [Adding SSL certificates signed by a root CA](#adding-ssl-certificates-signed-by-a-root-ca)
    - [Getting agent installers to work](#getting-agent-installers-to-work)
    - [Updating Tactical RMM breaks agent installers](#updating-tactical-rmm-breaks-agent-installers)
- [References](#references)

## Initial configuration

Follow this guide on [configuring a server running Debian](/servers/configuring-debian-server.md) to set up a new user account and secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

## Installing Tactical RMM

Use [the guide](https://docs.tacticalrmm.com/install_server/) provided by the Tactical RMM team. However, be sure to read the remainder of this guide ***before*** attempting the install, as it will prevent what seem to be "common errors."

## Troubleshooting a local deployment

There are handful of things that should be done in order to have everything work correctly in a local deployment.

### tmux

Install `tmux` and run the "tactical" user's command in the `tmux` shell. That way, if there are any issues with connection via `ssh`, the connection isn't dropped. If `tmux` is used, it will be possible to reattach to the install script and continue. Without `tmux`, it's possible to drop out of the install with no way back in. The only solution at this point is to reinitialize the VM and start over.

### Self-signed SSL certificates

When doing a local deployment that is ***not*** a production deployment, make sure to run the install script with the "--insecure" flag.

```
./install.sh --insecure
```

This will generate self-signed SSL certificates. Without this, the installation will fail. The only solution at this point is to reinitialize the VM and start over.

### locales

At the beginning of the install, the script may shout about `locales` not being set up properly. This is an irritating issue. The install script *specifically* wants the user to run the following:

```
sudo dpkg-reconfigure locales
```

Then, confirm that *only* "en_US.UTF-8" is selected. On the next screen, generate the locales using "en_US.UTF-8".

One may believe that editing the "/etc/locale.gen" file and then running `locale-gen` would result in the same outcome. For whatever reason, it does not.

### Domains and DNS

At the beginning of the install (after the potential "locales" issue), the script will ask the user to input domains for the different services. Use the following schema:

```
api         = api.domain.tld
rmm         = rmm.domain.tld
mesh        = mesh.domain.tld
domain      = domain.tld
user e-mail = $EMAIL
```

"domain.tld" may look like the following:

```
$HOSTNAME.domain.tld
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

### Connect to the dashboard

To use the example from before, https://tactical-rmm.example.com will bring up the dashboard login. However, using this link *will* result in a "backend is offline" error message.

Therefore, be sure to access the dashboard from https://rmm.tactical-rmm.example.com URL.

This should work, and will direct the user to input a TOTP code.

### Removing the insecure flag

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

### Adding SSL certificates signed by a root CA

In conjunction with step six, an additional measure can be taken to secure the server with SSL certificates that have been signed by a root CA (certificate authority).

Follow the guide in this Git repo about creating and setting up certificates.

The one key detail to note is that Tactical RMM will require a wildcard certificate. However, creating a wildcard certificate for `*.example.com` will not work, as the wildcard only seems to work on a single domain level above the listed domain.

Therefore, a wildcard certificate would need to be created for `*.tactical-rmm.example.com`. Certificates made with this as the listed domain will be approved by the root CA.

Simply replace the "cert.pem" (public key) and "key.pem" (private key) with those found in the "/etc/ssl/tactical" directory. Then, restart the server. So long as the root CA has been added to the machine accessing the web UI, either at the browser or the OS level, the certificate will register as valid.

### Getting agent installers to work

After swapping out the public and private keys in the server's "/etc/ssl/tactical" directory, the server's web UI will now be accessible. However, any attempt to create an agent installer, and install it on a client computer, will fail with the error "unable to download the mesh agent from the RMM." This is simply due to server failing to find a root CA that can verify the SSL certificates in the "/etc/ssl/tactical" directory. Therefore, the solution is to provide the root CA to the RMM software.

Adding the root CA to the server's certificate store can be achieved a few different ways. Either way will work.

- One way to do this is to add the root CA (public key) directly to the file "/etc/ssl/certs/ca-certificates.crt".
- Another way would be to add the "CA.crt" file to any subdirectory of "/usr/local/share/ca-certificates" -- a suggested "best practice" would be to create a directory within "/usr/local/share/ca-certificates" that specifies the issuer of the CA (ex. "company-CA"), and then moving the "CA.crt" file into that subdirectory.
    - If the file is added to "/usr/local/share/ca-certificates", then the command `sudo update-ca-certificates` must be run, in order to update the certificate store.

However, adding the root CA to the one of those two directories is not enough, as the RMM software does not know to query the server's certificate store for the "CA.crt". In order to enable the RMM to query the server's certificate store, an additional set of steps must be performed. The following is thanks to user MrGoodbody on [this GitHub issue](https://github.com/amidaware/tacticalrmm/discussions/1114).

As the user "tactical", run the following commands:

```
source /rmm/api/env/bin/activate
pip install pip-system-certs
systemctl restart rmm
```

Performing these commands will install a Python package that will enable the RMM software to query the server (after restarting the RMM) and obtain the "CA.crt" credentials needed to verify the SSL certificates. Once this is done, the agent installers will no longer throw the error message, and Tactical RMM will be fully operational.

### Updating Tactical RMM breaks agent installers

Sometimes, but not always, running the "update.sh" script will break the agent installer functionality. This is because the update script will reset what certificates are accessed by the server.

If this happens, simply repeat the steps from [Getting agent installers to work](#getting-agent-installers-to-work), specifically:

```
source /rmm/api/env/bin/activate
pip install pip-system-certs
systemctl restart rmm
```

This will update `pip-system-certs`, if necessary, and will add those certificates back into the server's certificate store.

As of current testing, this has always resolved the problem.

---

Congrats on getting the Tactical RMM server deployed!

## References

- [Tactical RMM's official guide on installing a Tactical RMM server](https://docs.tacticalrmm.com/install_server/)
- [GitHub - Tactical RMM Issue #1114 - Unable to download the mesh agent from the RMM](https://github.com/amidaware/tacticalrmm/discussions/1114)
    - MrGoodbody's response was the key to fixing agent installers
