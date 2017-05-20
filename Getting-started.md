The following information will assist you with the installation of [Alpine Linux].

[[_TOC_]]

## Installation Quick-Start in 3 Easy Steps

1. [Download] the latest stable-release ISO.
2. If you have a CD drive from which you can boot, then burn the ISO onto a 
blank CD using your favorite CD burning software. Else [[create a bootable USB drive]].
3. Boot from the CD or USB drive, login as root with no password, and voilà! Enjoy [Alpine Linux]!

## Installation Handbook

### Basics

Alpine Linux can be used in any of the following three modes

#### Diskless mode

You'll boot from read-only medium such as the installation CD, a USB drive, or a Compact Flash card.

When you use Alpine in this mode, you need to use Alpine Local Backup (lbu) to save your modifications between reboots. That requires some writable medium, usually removable. (If your boot medium is, for example, a USB drive, you can save modifications there; you don't need a separate partition or drive.) See also Local APK cache.

#### Data mode

As in diskless mode, your OS is run from a read-only medium. However, here a writable partition (usually on a hard disk) is used to store the data in /var. That partition is accessed directly, rather than copied into a tmpfs; so this is better-suited to uses where large amounts of data need to be preserved between reboots.

#### Sys mode

This is a traditional hard-disk install (see link for details). Both the boot system and your modifications are written to the hard disk, in a standard Linux hierarchy.

[Alpine Linux]: https://alpinelinux.org
[Download]: https://alpinelinux.org/downloads