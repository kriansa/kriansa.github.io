---
layout: post
title: 'The case of the mysterious XFS corruption: A NIC swap gone wrong'
date: 2024-12-24 23:39 -0300
---

What started as a simple network card upgrade on my home server turned into quite the
troubleshooting adventure. I thought I'd share this experience since it taught me some valuable
lessons about PCI-E passthrough in virtualized environments, and might save someone else a few hours
of debugging.

I was just swapping out a Realtek NIC for an Intel I226V - a straightforward hardware upgrade that
should have taken 15 minutes tops. After cleaning out some dust and installing the new card in the
same slot as the old one, I powered up the server. That's when things got interesting.

The first sign of trouble was the HDD activity LED - instead of its usual blinking pattern
indicating normal I/O activity, it stayed constantly lit after booting into Proxmox. Then I noticed
none of my services were coming back online. When I checked the IPMI interface, I was greeted with a
barrage of XFS errors:

``` 
XFS (dm-0): metadata I/O error in "xfs_da_read_buf+0xf4/0x150 [xfs]" at daddr 0x1816a90 len 8 error 5 
XFS (dm-0): metadata I/O error in "xfs_da_read_buf+0xf4/0x150 [xfs]" at daddr 0x1816a90 len 8 error 5 
XFS (dm-0): metadata I/O error in "xfs_da_read_buf+0xf4/0x150 [xfs]" at daddr 0x1816a90 len 8 error 5 
XFS (dm-0): metadata I/O error in "xfs_imap_to_bp+0x5c/0x80 [xfs]" at daddr 0x8a66c0 len 32 error 5 
```

Following these errors, the filesystem would shut down and make the system unoperable:

``` 
XFS (dm-0): log I/O error -5 XFS (dm-0): Filesystem has been shut down due to log error (0x2).
XFS (dm-0): Please unmount the filesystem and rectify the problem(s). 
```

## A Christmas eve troubleshooting adventure

My first thought was filesystem corruption. I booted from an Arch Linux live USB and ran
`xfs_repair -v` on the root partition. Surprisingly, it found no issues. The drive's SMART data also
showed everything was fine. I rebooted and faced the same issue. Something wasn't adding up.

The breakthrough came when I tried booting in safe mode - suddenly everything worked perfectly. I
started the `pveproxy` service to get access to the Proxmox UI, and things were still stable. But as
soon as I tried to start my TrueNAS VM - boom - the same XFS errors appeared and I had to hard reset
the system.

This was the clue I needed. Why would starting a specific VM cause host filesystem errors? I
disabled auto-start for all VMs and began testing them one by one. Other VMs worked fine; the
problem was specific to my TrueNAS VM. Even a clone of the VM showed the same behavior.

## The root cause

Finally, I checked the PCI-E passthrough configuration for the TrueNAS VM, and there it was -
somehow, during the NIC swap, the VM had been configured to passthrough the host's NVME boot disk
instead of the SAS controller it was supposed to use. When the VM tried to take exclusive control of
the boot drive, the host filesystem naturally went haywire.

The fix was trivial once I found the actual problem - I just corrected the passthrough configuration
to use the correct address to the SAS controller, and everything went back to normal. But this
experience taught me that hardware changes can have unexpected ripple effects on PCI-E device
mappings, and what looks like filesystem corruption might actually be something completely
different.

## A note on PCI-E passthrough

PCI-E passthrough has always been one of those finicky technologies that keeps system administrators
on their toes. We're all familiar with the common gotchas: making sure IOMMU is properly enabled,
dealing with device groups, handling reset bugs, and so on. But here's a new one to add to the list:
hardware changes, even seemingly unrelated ones like swapping a network card, can unexpectedly alter
PCI-E device addresses and mappings. 

This means that after any hardware change - whether it's adding a new card, removing an old one, or
even just moving devices between slots - it's crucial to verify your PCI-E passthrough
configurations. Don't assume they'll remain correct just because you didn't directly touch the
devices being passed through. The few minutes spent double-checking these configurations could save
you hours of debugging mysterious system errors that seem to point in completely wrong directions.

## A note on XFS error handling

Looking back at this incident, I'm actually grateful for how XFS handled this error condition. By
immediately detecting the I/O errors and shutting down the filesystem, it prevented what could have
been a catastrophic scenario. Imagine if the filesystem had continued operating while two different
systems (the host and the VM) were trying to access the same physical drive simultaneously - we
could have ended up with corrupted data or, worse, completely trashed filesystems on both ends. The
aggressive shutdown behavior that initially seemed frustrating was actually a critical safety
mechanism that prevented me from shooting myself in the foot.

If your filesystem starts throwing errors and shutting down - thank it for potentially saving your
data before diving into debugging! It might save you a few hours of debugging, or in this case,
possibly days of data recovery.
