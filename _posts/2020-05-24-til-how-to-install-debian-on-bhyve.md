---
layout: post
title: 'TIL: How to install Debian on bhyve (or FreeNAS)'
date: 2020-05-24 17:10:00 BRT
tags: linux, debian, nfs, til
categories: articles
---

When installing Debian 10 on FreeNAS (which uses bhyve as the hypervison) using UEFI, I faced some
issues whereas once I removed the installation media, I couldn't boot anymore and I went straight to
the EFI shell.

After some digging, I found this [awesome article][article] where KrisBee explains the root cause of
this issue.

The fact is that Debian's GRUB EFI installation relies on a EFI-standard that enforces a separation
of boot entries by keeping them in separate folders/names on the _ESP_ (EFI system partition) - in
this case `EFI/debian/grubx64.efi`. Unfortunately, bhyve doesn't seem to [support that
yet][bhyve-support], and it expects the bootloader to be at `EFI/BOOT/bootx64.efi`.

To work around that, we need to force Debian installation to use the removable media path, which
basically means it will install the boot entry at `EFI/BOOT/bootx64.efi`. To enable that, we need to
either use the Expert install or the Rescue mode available on the installation media.

![debian 10 on bhyve](/assets/images/deb10_bhyve.jpeg)

Lastly, it's worth mentioning that after removing the CD-ROM media from the VM through FreeNAS, the
order of the devices will change and you will lose network connectivity. You will need to get the
interface name and set it to `/etc/network/interfaces`.

[article]: https://www.ixsystems.com/community/threads/howto-how-to-boot-linux-vms-using-uefi.54039/page-9#post-516980
[bhyve-support]: https://github.com/churchers/vm-bhyve/issues/336#issuecomment-573731049
