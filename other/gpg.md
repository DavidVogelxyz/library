# GPG

NB: What some people refer to as "private keys", `gpg` refers to as "secret keys". They are one and the same.

## Table of contents

- [Creating a GPG key](#creating-a-gpg-key)
- [Listing keys](#listing-keys)
- [How to encrypt a file](#how-to-encrypt-a-file)
- [Exporting keys](#exporting-keys)

## Creating a GPG key

Create a GPG key with:

```
gpg --full-generate-key
```

## Listing keys

List all public keys with:

```
gpg --list-public-keys
```

List all private keys with:

```
gpg --list-secret-keys
```

List public key fingerprints with:

```
gpg --fingerprint --fingerprint
```

Selecting the different key ID formats:

```
gpg --list-public-keys --keyid-format $KEY_ID_FORMAT
```

Key ID format options:
- none
- short
- long
- 0xshort
- 0xlong

Explanations:

- A short key ID is the last 8 characters of the fingerprint.
- A long key ID is the last 16 characters of the fingerprint.
- An '0x' key ID prefixes the key with '0x' to denote that the key ID is a hex value.

## How to encrypt a file

Encrypt a file with the following:

```
gpg --encrypt --armor --recipient $RECIPIENT_GPG_FINGERPRINT --output $OUTPUT_FILENAME $INPUT_FILENAME
```

Notes:

- "--encrypt" is self-explanatory.
- "--armor" encrypts the message into ASCII armor, outputting it into text that can be shared easily.
- "--recipient" enables the file to be encrypted such that only the individual with the correct private key can decrypt the message.
    - Without this option, only the private key of the one encrypting the message can be used to decrypt the message.
- "--output" enables the user to give the output a specific name. Otherwise, the default is "$INPUT_FILENAME.gpg".
    - As an example, this option allows the user to output the file to a ".asc" filetype.

Decrypt a message with the following:

```
gpg --decrypt $INPUT_FILENAME --output $OUTPUT_FILENAME
```

Notes:

- "--output" enables the user to output the decrypted file to a specific filename, rather than to STDOUT.

## Exporting keys

NB: If no $KEY_ID is specified, then all keys will be exported. This is true for both public and secret keys.

Export a public key:

```
gpg --export $KEY_ID --armor --output $OUTPUT_FILENAME
```

Export a secret key:

```
gpg --export-secret-keys $KEY_ID --armor --output $OUTPUT_FILENAME
```

Import a key:

```
gpg --import $KEY_FILE
```

Delete a key:

```
gpg --delete-key $KEY_ID
```
