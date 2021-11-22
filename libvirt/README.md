# Finnix libvirt examples

This directory contains a number of example libvirt definitions for Finnix on various architectures and in various configurations.  To import one, for example:

```
virsh define finnix-amd64-uefi.xml
```

These profiles assume an amd64 host, and will virtualize amd64/i386 and emulate the other architectures.

For direct kernel/initrd machines, the images are extracted directly from the ISO.

On riscv64, `riscv64-opensbi-fw_jump.elf` is taken from the Ubuntu [opensbi package](https://packages.ubuntu.com/search?keywords=opensbi), 21.10 (impish) required as of this writing, even if the host qemu OS is 20.04 LTS.