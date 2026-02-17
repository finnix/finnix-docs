# Finnix checksums and signatures

Data in the [releases directory](releases/) contains a number of checksums and signatures for release files.

## OpenPGP signatures

Releases are signed by the OpenPGP key "Finnix Release Signing Key &lt;<keymaster@finnix.org>&gt;".

```shell
gpg --recv-keys 867279B83BF8E815A236D39E7D6F85C04356E6C2
gpg --verify finnix.iso.gpg finnix.iso
```

Successful output should look similar to:

```text
gpg: Signature made using RSA key 867279B83BF8E815A236D39E7D6F85C04356E6C2
gpg: Good signature from "Finnix Release Signing Key <keymaster@finnix.org>"
```

## SSH signatures

Releases are signed by an ED25519 SSH key.  Create a file called `authorized_signers`:

```text
file-signing@finnix.org ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJQEw5EJeL+lHS9Dq8iU8vgVgU6VAb+sVsitQU8tHl38 Finnix file-signing
```

Then run:

```shell
ssh-keygen -Y verify -f authorized_signers -I file-signing@finnix.org -n file -s finnix.iso.sig < finnix.iso
```

Successful output should be:

```text
Good "file" signature for file-signing@finnix.org with ED25519 key SHA256:8e25ds3mwVLnebopr3lltmpABttlnK0JkeVLduz/43s
```

## Checksums

SHA512, SHA256 and MD5 checksums are provided.  If using these to verify transfer of a release, SHA512 *and* SHA256 should both be checked.  MD5 is provided for historical completeness only and should not be relied upon.

Historical note: The MD5 checksums for Finnix ISOs between versions 100 and 111 all start with "f${version}ff", e.g. finnix-105.iso's MD5 checksum is "f105ff476a228d05d663fc52bd197721".  This was produced by a utility called [vanityhash](https://github.com/rfinnie/vanityhash) which searches a checksum namespace by appending trash data to the file. Finnix 120 and onward no longer do this, because the hybrid BIOS/UEFI booting relies on data at the end of the image.

## License

This document is provided under the following license:

    SPDX-PackageName: finnix-docs
    SPDX-PackageSupplier: Ryan Finnie <ryan@finnie.org>
    SPDX-PackageDownloadLocation: https://github.com/rfinnie/finnix-docs
    SPDX-FileCopyrightText: © 2021 Ryan Finnie <ryan@finnie.org>
    SPDX-License-Identifier: CC-BY-SA-4.0
