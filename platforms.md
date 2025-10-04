# Platform support on Finnix

## AMD64

From Finnix 120 onward, AMD64 is the primary and only officially supported architecture. All 64-bit x86 systems going back to 2003 should be bootable by Finnix.  The latest Finnix release may be downloaded from [finnix.org](https://www.finnix.org/).

## i386

Finnix versions 111 and earlier supported a 32-bit i386 userland, with most releases having both 32-bit and 64-bit kernels. (While AMD64 CPUs are capable of booting 32-bit kernels, memory was limited to 32-bit spaces.)

Note that "i386" refers to the 32-bit x86 overall architecture; no Finnix version has ever supported 386 or 486 CPUs.

While i386 Finnix releases are no longer being made, if you need to boot Finnix on an older 32-bit only x86 CPU, [Finnix 109](https://www.finnix.org/releases/109/finnix-109.iso) is recommended. (Finnix 110 and 111 supported i386 but were not as stable as Finnix 109.)

See also below about i386 as a buildable current architecture.

## PowerPC

As with i386 above, most releases from Finnix 86.1 through 111 included PowerPC ISOs.  This was a 32-bit userland with 32-bit and 64-bit kernels, the latter supporting G5 processors.

No currently supported mainstream Linux distribution officially supports Apple-era PowerPC anymore. Like i386, [Finnix 109 (PowerPC)](https://www.finnix.org/releases/109/finnix-ppc-109.iso) is recommended for these systems.

## Buildable current architectures

While AMD64 is the only architecture currently supported by Finnix and the only architecture with ISOs being released, [finnix-live-build](https://github.com/finnix/finnix-live-build), the system which produces Finnix builds is capable of building images for a number of architectures.

As of this writing, these architectures are:

* amd64
* i386
* arm64
* armhf
* ppc64el (not to be confused with the older PowerPC architecture)
* s390x
* riscv64

Please see the finnix-live-build README for details about these architectures.

## Apple platforms

Apple platforms are generally a moving target, with the rule of thumb for Linux distributions (Finnix included) being the best compatibility will be an older device with a newer distribution.

Finnix 109 (PowerPC) generally boots and functions well on Apple PowerPC systems, but keep in mind that Finnix 109 itself was released in 2013, and that no current Linux distributions support PowerPC anymore.

The latest version of Finnix (AMD64) should work well on all former Apple (Intel) systems.

Finnix for arm64 is not officially supported, but is buildable, see above. That being said, it is not bootable directly on Apple Silicon Macs. However, it is bootable within virtualization on an Apple Silicon system, assuming the virtualization software emulates a UEFI system. This configuration has been tested with [UTM](https://mac.getutm.app/).

## License

This document is provided under the following license:

    SPDX-PackageSummary: finnix-docs
    SPDX-FileCopyrightText: © 2021 Ryan Finnie <ryan@finnie.org>
    SPDX-License-Identifier: CC-BY-SA-4.0
