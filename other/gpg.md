# GPG

NB: What some people refer to as "private keys", `gpg` refers to as "secret keys". They are one and the same.

## Table of contents

- [Creating a GPG key](#creating-a-gpg-key)
- [Listing keys](#listing-keys)
- [How to encrypt a file](#how-to-encrypt-a-file)
- [Exporting keys](#exporting-keys)
- [Changing the passphrase to a GPG key](#changing-the-passphrase-to-a-gpg-key)
- [References](#references)

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
gpg --armor --output $OUTPUT_FILENAME --export $KEY_ID
```

Export a secret key:

```
gpg --armor --output $OUTPUT_FILENAME --export-secret-keys $KEY_ID
```

Import a key:

```
gpg --import $KEY_FILE
```

Delete a key:

```
gpg --delete-key $KEY_ID
```

## Changing the passphrase to a GPG key

To change the passphrase to a GPG private key, use the following command:

```
gpg --edit-key $KEY_ID
```

After using the above command to enter the "edit key" menu, use the following command to change the passphrase:

```
passwd
```

Once the passphrase has been changed, use `save` to confirm the changes and exit the GPG key menu.

There are a few additional things to note about the use of this command:

- The key ID for the private key must be specified
- The key must be stored in the keyring
    - It is not possible to use this command to change the passphrase on a key that's been exported using `gpg --export-secret-keys $KEY_ID`

## References

- [Medium - GPG Quickstart Guide](https://medium.com/@acparas/gpg-quickstart-guide-d01f005ca99)
    - Original reference for how to create GPG keys and how to use them properly
- [CyberCiti - GPG Change Passphrase Secret Key Password Command](https://www.cyberciti.biz/faq/linux-unix-gpg-change-passphrase-command/)
    - Reference for changing the passphrase on a GPG key
