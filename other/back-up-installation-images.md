# Backing up installation images

## Introduction

Knowing how to back up data is extremely useful. While there are many solutions for backing up files, backing up computers (as a whole) is generally thought to be a much more difficult task. However, most Linux distributions come with coreutils and built-in commands that make it extremely easy to back up entire systems for redeployment.

This guide shares some of those methods.

## Table of contents

- [Using dd](#using-dd)
    - [Using sparse conversion with dd](#using-sparse-conversion-with-dd)
- [Using gzip and zcat](#using-gzip-and-zcat)
- [Notes from "old ways"](#notes-from-"old-ways")
- [References](#references)

## Using dd

`dd` is a coreutil of GNU, accessible by nearly every Linux installation, that allows the user to copy inputs to outputs. By making use of `dd`, it is possible to create a bit-for-bit copy of any drive, which makes it perfect as a backup utility. Often, this backup will be saved as a file ending in ".img" or ".iso".

Options used with `dd` include the following:

- `if=` specifies the input file.
- `of=` specifies the output file.
- `status=progress` will tell `dd` to show progress information as it runs.
- `conv=` converts the input file "as per the comma separated symbol list"
- `iflag=` tells `dd` to read input "as per the comma separated symbol list"
- `oflag=` tells `dd` to write output "as per the comma separated symbol list"
- `bs=` defines the block size that `dd` uses when writing the output

Do this using a live USB environment, as the device being backed up ***must be*** unmounted.

Create the file.img backup file with the following command:

```
dd if=/dev/$sdx of=file.img status=progress
```

Once the backup image is written, it can be deployed with the following command:

```
dd if=file.img of=/dev/$sdx status=progress
```

### Using sparse conversion with dd

It is also possible to use `dd` in a way that incorporates data compression, just like `gzip`. To do this, add the `conv=sparse` option to the `dd` command, as in the following example:

```
dd if=/dev/$sdx of=file.img conv=sparse status=progress
```

In addition, as is described in [this ServerFault post](https://serverfault.com/questions/665335/what-is-fastest-way-to-copy-a-sparse-file-what-method-results-in-the-smallest-f), the following command gives the best overall result in terms of time-to-completion and output quality (based on benchmark testing).

```
dd if=/dev/$sdx of=file.img iflag=direct oflag=direct bs=256k conv=sparse status=progress
```

## Using gzip and zcat

`gzip` is a data compression program that allows the user to copy inputs to outputs, similar to `dd`. However, the key difference is that `gzip` is able to create smaller output files than `dd`, which can be incredibly useful when making backups of system installtions. This is especially true when the system being backed up has paritions containing large amounts of unwritten storage space.

Do this using a live USB environment, as the device being backed up ***must be*** unmounted.

Create the file.img backup file with the following command:

```
gzip -v9c /dev/$sdx > file.img.gz
```

The options are as follows:

- 'v' is a verbosity setting. When writing to a file using '>', the verbosity setting doesn't appear to display. But, keep it in anyway, as good habit.
- '9' is a setting that opts for best compression quality, at the expense of speed.
- 'c' is a flag that tells `gzip` to keep the original 'file' instead of deleting it.

Once the backup image is written, it can be deployed with the following command:

```
zcat file.img.gz > /dev/$sdx
```

### Additional comments on zcat

I was taught to deploy the backup with `zcat file.img.gz > /dev/$sdx`. However, `dd` can also be used to stream the output data:

```
zcat file.img.gz | dd of=/dev/$sdx
```

One benefit of piping the output of `zcat` into `dd` is that it allows for the use of the `status=progress` switch:

```
zcat file.img.gz | dd of=/dev/$sdx status=progress
```

## Notes from "old ways"

Originally, I learned to write these images by using `cat`, like so:

```
cat /dev/$sdx | gzip -v9c > file.img.gz
```

Similarly, `dd` can also be used to read the input file:

```
dd if=/dev/$sdx | gzip -v9c > file.img.gz
```

However, `gzip` can be run directly on files. It is not necessary to run `cat` or `dd` on the device in order to pipe it into `gzip`. This is identical to Luke Smith's rant about "[don't `cat` into `grep`](https://www.youtube.com/watch?v=82NBMvx6vFY)"

## References

- [ServerFault - What is fastest way to copy a sparse file? What method results in the smallest file?](https://serverfault.com/questions/665335/what-is-fastest-way-to-copy-a-sparse-file-what-method-results-in-the-smallest-f)
- [YouTube - Luke Smith - Don't cat into grep](https://www.youtube.com/watch?v=82NBMvx6vFY)
- [YouTube - anthonywritescode - don't use cat! (intermediate) anthony explains #508](https://www.youtube.com/watch?v=vWMiBVkdJjA)
