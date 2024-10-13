# Finnix kernel command line options

Finnix supports a number of kernel command line options. They may be specified by:

* UEFI GRUB: Press "e" on "Live system", go down to the "linux" line and append to the end of the line. When ready, press Ctrl-X to boot.
* BIOS isolinux: Press Tab on "Live (amd64)" and append to the end of the line. When ready, press Enter to boot.

## Early boot

Finnix uses the Debian live-boot system to provide early boot functionality, and live-boot supports a number of kernel command line options, such as manual IP address configuration via `ip=`. Please see the [live-boot manpage](https://manpages.debian.org/testing/live-boot-doc/live-boot.7.en.html) for a list of supported options.

## systemd options

Likewise, systemd supports a number of kernel command line options, though many are not applicable to a live environment.  See [its manpage](https://www.freedesktop.org/software/systemd/man/kernel-command-line.html) for a list of options.

## Finnix options

The following kernel command line options are specific to Finnix:

* `0`: Start locale (keyboard and language) configuration during early boot. This is the same as running the `0` command after boot. (`0` is chosen as it is hopefully as keyboard-agnostic of a key as possible before the keyboard layout can actually be specified).
* `sshd`: Start the SSH daemon upon boot.
* `passwd`: Set account passwords on the kernel command line.  For example:
  * `passwd=foo` - Implicit root user, plain password "foo"
  * `passwd=root:foo passwd=finnix:bar` - Set the root user to "foo" and the finnix user to "bar"
  * `passwd=root:$1$QnvsFWnz$MppFa1JL0xsMjB/VQwvwv.` - Explicit root user, hashed password
