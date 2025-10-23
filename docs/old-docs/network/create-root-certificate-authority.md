Creating and configuring a root certificate authority
=====================================================

Introduction
------------

A certificate authority (CA) is an entity that signs digital certificates, such as SSL certificates. Often, when creating public servers and services, a package such as `certbot` is used to generate certificates signed by Let's Encrypt, a non-profit CA run by the Electronic Frontier Foundation (EFF).

When creating services to run on a local network, it's possible to use self-signed certificates. An issue that arises from using self-signed certificates is that the webpages secured by those certificates will display a warning that must be clicked through.

By using a root CA, those warnings can be bypassed. The root CA can issue a root certificate to be uploaded into a browser or operating system, which basically says "any certificate signed by the root CA is legitimate." Then, by having the root CA sign the certificates, any computer that has the root certificate in its certificate stores will accept the self-signed certificate and bypass the warning page.

This guide provides instructions for creating certificates signed by a self-generated root CA.

Table of contents
-----------------

- [Introduction](#introduction)
- [Creating a root CA](#creating-a-root-ca)
- [Creating certificates](#creating-certificates)
    - [Single server certificate](#single-server-certificate)
    - [Wildcard server certificate](#wildcard-server-certificate)
- [Reading a certificate file](#reading-a-certificate-file)
- [References](#references)

Creating a root CA
------------------

First, create the private key that will be used by the root CA.

```
openssl genrsa -out rootCA.key 4096
```

Next, create a root CA certificate, and sign it with the private key created by the previous command.

```
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 36500 -out rootCA.crt
```

Creating certificates
---------------------

There are a few different types of certificates that can be created with a root CA, including single server certificates and wildcard certificates.

A single server certificate is a great option because of its security -- a new certificate gets created for every server, and therefore it is easier to revoke and redeploy a compromised certificate. The drawback is one of convenience -- because of the lack of use of wildcards, a new certificate has to be created for every server deployed on the network.

A wildcard certificate is able to secure all subdomains below a higher level domain. A single server certificate can be deployed on multiple servers within a domain. The benefits should be obvious -- creating a single certificate that can be added to any number of servers drastically reduces the workload that would come with creating a different certificate for every service. The drawback is one of security -- if compromised, that certificate can be used to attack the network. If a compromised certificate is deployed on a hostile machine, then it can appear to be a legitimate service on the network. This would give an adversary the ability to further attack users on the network.

### Single server certificate

To create a single server certificate, first create the private key that will be used to sign the certificates. Note that `<SERVER>` should be changed to the name of the server that will use this key.

```
openssl genrsa -out <SERVER>.key 4096
```

Next, use the following command to create the certificate signing request (CSR) file. Note the following:

- `<SERVER>` is the name of the server that will use this key and be obtaining its CSR. **This needs to reflect the name of the key chosen in the previous command.**
- `C` is the country where the server resides. The guide assumes a server located in the US. This can be left blank.
- `<STATE>` is the state within the US where the server resides. This can be left blank.
- `<ORGANIZATION>` is the organization (or company/entity) that owns the server. This can be left blank.
- `<ORGANIZATIONAL_UNIT>` is the department with the organization that controls the server. This can be left blank.
    - Note: if the Organizational Unit is multiple words, those words do not need to be surrounded by quotes. Anything that comes before the `/CN` will be taken as the value for `OU`.
- `CN` is the common name of the server, and should be the FQDN of the server, without a subdomain.
    - Subdomains can be specified as subject alternative names (SANs).
- `<DOMAIN.TLD>` is the FQDN of the server. **This option should *not* be left blank!**
- If the CSR will have multiple SANs, the line starting with `-config` should be formatted as follows:
    - `-config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:<SERVER>.<DOMAIN.TLD>\nsubjectAltName=DNS:<SUBDOMAIN>.<SERVER>.<DOMAIN.TLD>")) \`

```
openssl req -new -sha256 -key <SERVER>.key \
    -subj "/C=US/ST=<STATE>/O=<ORGANIZATION>/OU=<ORGANIZATIONAL_UNIT>/CN=<DOMAIN.TLD>" \
    -reqexts SAN \
    -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:<SERVER>.<DOMAIN.TLD>")) \
    -out <SERVER>.csr
```

Finally, use the next command to create the single server's certificate.

Note the following:

- As before, `<SERVER>` needs to be swapped to the name of the server that was used to name the key and the CSR file in the previous commands.
- `<DOMAIN.TLD>` is the FQDN of the server. This should be the same FQDN from the previous command.
- If the certificate will have multiple SANs, the line starting with `-extfile` should be formatted as follows:
    - `-extfile <(printf "subjectAltName=DNS:<SERVER>.<DOMAIN.TLD>,DNS:<SUBDOMAIN>.<SERVER>.<DOMAIN.TLD>") \`
        - These should be the same SANs as the ones listed in the CSR.

```
openssl x509 -req -sha256 \
    -days 36500 \
    -extfile <(printf "subjectAltName=DNS:<SERVER>.<DOMAIN.TLD>") \
    -in <SERVER>.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial \
    -out <SERVER>.crt
```

Add the key and certificate to the server.

### Wildcard server certificate

To create a wildcard certificate, first create the private key that will be used to sign the certificates.

```
openssl genrsa -out server.key 4096
```

Next, use the following command to create the certificate signing request (CSR) file. Note the following:

- `C` is the country where the server resides. The guide assumes a server located in the US. This can be left blank.
- `<STATE>` is the state within the US where the server resides. This can be left blank.
- `<ORGANIZATION>` is the organization (or company/entity) that owns the server. This can be left blank.
- `<ORGANIZATIONAL_UNIT>` is the department with the organization that controls the server. This can be left blank.
    - Note: if the Organizational Unit is multiple words, those words do not need to be surrounded by quotes. Anything that comes before the `/CN` will be taken as the value for `OU`.
- `CN` is the common name of the server, and should be the FQDN of the server, without a subdomain or a wildcard.
    - Subdomains and wildcards can be specified as subject alternative names (SANs).
- `<DOMAIN.TLD>` is the FQDN of the server. **This is the only option that should *not* be left blank!**
- If the CSR will have multiple SANs, the line starting with `-config` should be formatted as follows:
    - `-config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:*.<DOMAIN.TLD>\nsubjectAltName=DNS:*.<SUBDOMAIN>.<DOMAIN.TLD>")) \`

```
openssl req -new -sha256 -key server.key \
    -subj "/C=US/ST=<STATE>/O=<ORGANIZATION>/OU=<ORGANIZATIONAL_UNIT>/CN=<DOMAIN.TLD>" \
    -reqexts SAN \
    -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:*.<DOMAIN.TLD>")) \
    -out server.csr
```

Finally, use the next command to create the wildcard server certificate.

Note the following:

- As before, `<SERVER>` needs to be swapped to the name of the server that was used to name the key and the CSR file in the previous commands.
- `<DOMAIN.TLD>` is the FQDN of the server. This should be the same FQDN from the previous command.
- If the certificate will have multiple SANs, the line starting with `-extfile` should be formatted as follows:
    - `-extfile <(printf "subjectAltName=DNS:*.<DOMAIN.TLD>,DNS:*.<SUBDOMAIN>.<DOMAIN.TLD>") \`
        - These should be the same SANs as the ones listed in the CSR.

```
openssl x509 -req -sha256 \
    -days 36500 \
    -extfile <(printf "subjectAltName=DNS:*.<DOMAIN.TLD>") \
    -in server.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial \
    -out server.crt
```

Add the key and certificate to the server.

An additional note on wildcard certificates: when a wildcard certificate is created, the wildcard appears to work for only those subdomains that are one level below the last FQDN.

- As an example, if the FQDN of `example.com` is specified, then the wildcard of `*.example.com` would work for any subdomain that is the third level down.
    - This would mean that FQDNs such as `subdomain.example.com` and `somehost.example.com` would both be covered by the wildcard certificate issued for `*.example.com`.
    - However, a FQDN such as `additional.subdomain.example.com` would not be covered by a wildcard certificate for `*.example.com`.

In order to get a wildcard certificate working for `additional.subdomain.example.com`, as well as another FQDN (like `another.subdomain.example.com`), the wildcard certificate would need to include `*.subdomain.example.com`. This is achieved through the use of subject alternative names (SANs).

Reading a certificate file
--------------------------

To print the contents of a certificate (ex. `<FILE.CRT>`), use the following command:

```
openssl x509 -text -noout -in <FILE.CRT>
```

References
----------

- [Baeldung - Creating a Self-Signed Certificate with OpenSSL](https://www.baeldung.com/openssl-self-signed-cert)
- [GitHub - fntlnz - Self-Signed Certificate with Custom CA](https://gist.github.com/fntlnz/cf14feb5a46b2eda428e000157447309)
