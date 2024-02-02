# Creating and configuring a root CA (certificate authority)

## Table of contents

- [Creating a root CA](#Creating-a-root-CA)
- [Single server certificate](#Single-server-certificate)
- [Wildcard server certificate](#Wildcard-server-certificate)

## Creating a root CA

First, create the private key that will be used to sign the root CA.

```
openssl genrsa -out rootCA.key 4096
```

Next, create a root CA, signing it with the private key (created by the previous command).

```
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 36500 -out rootCA.crt
```

## Single server certificate

This option is useful to create a single server certificate, to be deployed on a single subdomain.

This is a great option because of its security -- a new certificate gets created for every server, and therefore it is easier to revoke and redeploy a compromised certificate.

The drawback is one of convenience -- because of the lack of use of wildcards, a new certificate has to be created for every server deployed on the network.

To create a single server certificate, first create the private key that will be used to sign the certificates. Note that "[$SERVER]" should be changed to the name of the server that will use this key.

```
openssl genrsa -out [$SERVER].key 4096
```

Next, use the following command to create the CSR file.

Note the following:

- "[$SERVER]" is the name of the server that will use this key and be obtaining its CSR. **This needs to reflect the name of the key chosen in the previous command.**
- "C" is the country where the server resides. The guide assumes a server located in the US. This can be left blank.
- "[$STATE]" is the state within the US where the server resides. This can be left blank.
- "[$ORGANIZATION]" is the organization (or company/entity) that owns the server. This can be left blank.
- "[$ORGANIZATIONAL_UNIT]" is the department with the organization that controls the server. This can be left blank.
- "[$DOMAIN.COM]" is the FQDN of the server. **This option should *not* be left blank!**

```
openssl req -new -sha256 -key [$SERVER].key \
    -subj "/C=US/ST=[$STATE]/O=[$ORGANIZATION]/OU=[$ORGANIZATIONAL_UNIT]/CN=[$SERVER].[$DOMAIN.COM]" \
    -reqexts SAN \
    -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:[$SERVER].[$DOMAIN.COM]")) \
    -out [$SERVER].csr
```

Finally, use the next command to create the single server's certificate.

Note the following:

- As before, "[$SERVER]" needs to be swapped to the name of the server that was used to name the key and the CSR file in the previous commands.
- "[$DOMAIN.COM]" is the FQDN of the server. This should be the same FQDN from the previous command.

```
openssl x509 -req -sha256 \
    -extfile <(printf "subjectAltName=DNS:[$SERVER].[$DOMAIN.COM]") \
    -days 36500 \
    -in [$SERVER].csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial \
    -out [$SERVER].crt
```

Add the key and certificate to the server.

## Wildcard server certificate

This option is useful to create a single server certificate that can be deployed on multiple servers within a domain.

The benefits should be obvious -- creating a single certificate that can be added to any number of servers drastically reduces the workload that would come with creating a different certificate for every service.

The drawback is one of security -- if compromised, that certificate can be used to attack the network. If a compromised certificate is deployed on a hostile machine, then it can appear to be a legitimate service on the network. This would give an adversary the ability to further attack users on the network.

To create a wildcard certificate, first create the private key that will be used to sign the certificates.

```
openssl genrsa -out server.key 4096
```

Next, use the following command to create the CSR file.

Note the following:

- "C" is the country where the server resides. The guide assumes a server located in the US. This can be left blank.
- "[$STATE]" is the state within the US where the server resides. This can be left blank.
- "[$ORGANIZATION]" is the organization (or company/entity) that owns the server. This can be left blank.
- "[$ORGANIZATIONAL_UNIT]" is the department with the organization that controls the server. This can be left blank.
- "[$DOMAIN.COM]" is the FQDN of the server. **This is the only option that should *not* be left blank!**

```
openssl req -new -sha256 -key server.key \
    -subj "/C=US/ST=[$STATE]/O=[$ORGANIZATION]/OU=[$ORGANIZATIONAL_UNIT]/CN=*.[$DOMAIN.COM]" \
    -reqexts SAN \
    -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:*.[$DOMAIN.COM]")) \
    -out server.csr
```

Finally, use the next command to create the wildcard server certificate. Note that "[$DOMAIN.COM]" is the same FQDN from the previous command.


```
openssl x509 -req -sha256 \
    -extfile <(printf "subjectAltName=DNS:*.[$DOMAIN.COM]") \
    -days 36500 \
    -in server.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial \
    -out server.crt
```

Add the key and certificate to the server.

An additional note on wildcard certificates: when a wildcard certificate is created, the wildcard appears to work for only those subdomains that are one level below the last FQDN.

- As an example, if the FDQN of `example.com` is specified, then the wildcard of `*.example.com` would work for any subdomain that is the third level down.
    - This would mean that FQDNs such as `subdomain.example.com` and `somehost.example.com` would both be covered by the wildcard certificate issued for `*.example.com`.
    - However, a FQDN such as `additional.subdomain.example.com` would not be covered by a wildcard certificate for `*.example.com`.

In order to get a wildcard certificate working for `additional.subdomain.example.com`, as well as another FDQN (like `another.subdomain.example.com`), the wildcard certificate would need to be issued for `*.subdomain.example.com`.
