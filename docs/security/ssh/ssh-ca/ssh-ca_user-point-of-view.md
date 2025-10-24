The user's point of view
========================

[Back to the home page](README.md)

[Back to the previous page](getting-started-with-ssh-ca.md)

Table of contents
-----------------

- [Introduction](#introduction)
- [Additional details](#additional-details)

Introduction
------------

So, how is it possible that a user can authenticate to a host without any entries in the `authorized_keys` file? To explain, imagine a new user who joins an organization and requests access to an already configured SSH certificate authority. The process will look something like the following:
- A user will want access to some hosts governed by the SSH certificate authority.
- The user will request access from an administrator, and will provide the admin with a pubkey to a personal keypair that the user controls.
- The admin will sign a certificate on behalf of the user, combining the user's pubkey with a signature from the `user-CA_key`. This user certificate will allow access to hosts that are governed by the SSH CA infrastructure.
- The user will then use that certificate when logging into the host, in addition to their personal private key.

It's that simple!

Additional details
------------------

Some notes about user certificates are:
- The user certificate can be limited in many ways!
    - The user certificate can specify which user accounts the certificate is allowed to access, as well as which hosts are permitted.
    - The certificate can be given an expiration date, just like an SSL certificate, and the CA can also revoke that certificate at any time, using nothing more than the pubkey the user provided.
        - Just like with SSL, revocation of a user certificate will create a "revocation certificate" that must be shared with any and all hosts on which the admin team wants to revoke access.
- The user should be aware of how to utilize an SSH user certificate during the login process. This can be achieved in multiple ways:
    - To use a user certificate purely from the command line, the `ssh` command will look something like:
        - `ssh -i <PATH_TO_PRIVATE_KEY> -i <PATH_TO_USER_CERT> admin@mediaserver`
    - An SSH configuration file entry can also accept a certificate. This will look something like the following:

```
Host mediaserver
    HostName            mediaserver
    User                admin
    IdentityFile        <PATH_TO_PRIVATE_KEY>
    CertificateFile     <PATH_TO_USER_CERT>
```

To complete the user experience, the admin team may also provide the user with a host CA certificate, signed by the `host-CA_key`. This certificate can be added to the user's `known_hosts` file. The entry will look something like:

```
@cert-authority * ssh-ed25519-cert-v0a@openssh.com <CERTIFICATE> host-CA_key
```

The host CA certificate will allow the user to verify any and all hosts governed by the SSH certificate authority. Put another way, instead of having a `known_hosts` file entry for every host within the SSH certificate authority, one entry will be able to verify any and all hosts.

This is not only great from a "quality-of-life" standpoint, but also eliminates the pesky "the authenticity of host can't be established" messages. Said differently, users can be confident that hosts are valid servers -- any host that is not covered by the host CA certificate can be assumed to be ill-configured at best, and malicious at worst.

[Next page](ssh-ca_admin-point-of-view.md)
