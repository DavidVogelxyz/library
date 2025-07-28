# Cryptsetup

This guide serves the purpose of explaining how to manage an encrypted Linux partition after being configured during the initial OS install process.

## Table of contents

- [Changing the passphrase to an encrypted partition](#changing-the-passphrase-to-an-encrypted-partition)
- [Managing key slots](#managing-key-slots)
    - [Checking which key slots are in use](#checking-which-key-slots-are-in-use)
    - [Adding a key to an available key slot](#adding-a-key-to-an-available-key-slot)
    - [Adding a key to a specific key slot](#adding-a-key-to-a-specific-key-slot)
    - [Adding a key by using a keyfile](#adding-a-key-by-using-a-keyfile)
    - [Removing a key from a key slot](#removing-a-key-from-a-key-slot)
    - [Removing a key from a specific key slot](#removing-a-key-from-a-specific-key-slot)
- [What to do if an encryption passphrase has been forgotten](#what-to-do-if-an-encryption-passphrase-has-been-forgotten)
- [Dumping the master keys to an encrypted volume](#dumping-the-master-keys-to-an-encrypted-volume)

## Changing the passphrase to an encrypted partition

In order to change the passphrase to a LUKS encrypted partition, use the following command:

```
sudo cryptsetup luksChangeKey $PATH_TO_ENCRYPTED_VOLUME
```

The variable `$PATH_TO_ENCRYPTED_VOLUME` will look something like `/dev/sda2`, `/dev/nvme0n1p2`, or `/dev/mapper/$ENCRYPTED_VOLUME`. This is obviously dependent on how the encrypted partition was set up.

## Managing key slots

The `cryptsetup` package allows for multiple keys to be configured for an encrypted partition. To manage the different key slots, use the following instructions to do so.

### Checking which key slots are in use

Use the following command to see which key slots (0-7) are currently in use:

```
sudo cryptsetup luksDump $PATH_TO_ENCRYPTED_VOLUME | grep Slot
```

### Adding a key to an available key slot

To add another key to the next available key slot, use the following command:

```
sudo cryptsetup luksAddKey $PATH_TO_ENCRYPTED_VOLUME
```

Note: when `cryptsetup` asks for "any passphrase", it means "any passphrase that has been set using any of the key slots".

### Adding a key to a specific key slot

To add a key to a specific key slot, use the following command:

```
sudo cryptsetup luksAddKey $PATH_TO_ENCRYPTED_VOLUME -S $SLOT_NUMBER
```

### Adding a key by using a keyfile

To add a key by using a keyfile, use the following command:

```
sudo cryptsetup luksAddKey $PATH_TO_ENCRYPTED_VOLUME $PATH_TO_KEYFILE
```

### Removing a key from a key slot

To remove a key, use the following command:

```
sudo cryptsetup luksRemoveKey $PATH_TO_ENCRYPTED_VOLUME
```

The `cryptsetup luksRemoveKey` command will ask for a passphrase, and will remove the key corresponding to the passphrase entered.

### Removing a key from a specific key slot

To remove a key from a specific key slot, use the following command:

```
sudo cryptsetup luksKillSlot $PATH_TO_ENCRYPTED_VOLUME $SLOT_NUMBER
```

## What to do if an encryption passphrase has been forgotten

In most cases, if the encryption passphrase has been forgotten, the user is "SOL".

However, if the passphrase has been forgotten, but the encrypted volume is currently unlocked, it is possible to remedy the situation.

This can be achieved by following these instructions:

First, use the following command to assess which volume is the encrypted volume to be worked on:

```
sudo df -h
```

Once the encrypted volume has been identified, use the following command to show hashed versions of the key:

```
dmsetup table --showkeys
```

Copy the value of the hashed password to be modified. The hash should start with `SHA256`; copy the value that follows that header. Create a new file (e.g., `lukskey`) and paste the value into that new file.

Now, use the following command to read the file, convert the hash into binary format, and output it into a new `.bin` file:

```
sudo xxd -r -p lukskey lukskey.bin
```

As a note, the `-p` flag represents "postscript".

The following command will allow the user to add a new passphrase to the encrypted volume, using the `lukskey.bin` file as the current passphrase.

```
sudo cryptsetup luksAddKey $PATH_TO_ENCRYPTED_VOLUME --master-key-file <(cat lukskey.bin)
```

In the same way as the above, the following command should allow the user to modify the passphrase contained by the binary file, again using the `lukskey.bin` as the current passphrase.

```
sudo cryptsetup luksChangeKey $PATH_TO_ENCRYPTED_VOLUME --master-key-file <(cat lukskey.bin)
```

## Dumping the master keys to an encrypted volume

If, for whatever reason, the user wants to dump the master keys of an encrypted volume into a file, the user can perform the following command to do so:

```
sudo cryptsetup luksDump --dump-master-key $PATH_TO_ENCRYPTED_VOLUME
```

This command will dump the master key information into the terminal. This output can be directed, if desired, into a file. However, this contains the information required to unlock the encrypted volume, so extreme caution should be taken when using this command.
