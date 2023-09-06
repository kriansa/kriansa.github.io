---
layout: post
title: Creating a memory-backed storage on Linux
date: 2023-09-06 4:00:00 BRT
tags: linux, tmpfs
categories: articles
---
Frequently you'll find that `tmpfs` is the most convenient one. Alternatively there's also `ramfs`
and `ramdisk`.

Starting with `ramfs`, it's a memory FS that when mounted, it can use up to all available RAM space,
so you might beware of using it. It is not configurable. `tmpfs` extends from it, adding
configuration such as size limit. It by default allows swapping to disk when the system is under
memory pressure, however, it can be configured to not to. Both use the system read cache, and grow
or shrinks to accomodate the data in it. Setting them up is simple as mounting a filesystem, and no
prepare steps are necessary.

Ramdisk on the other hand, is a block device and you need to put a filesystem on top of it. It
allocates all the needed space upfront and is not dynamically resizable (i.e. you will need to
recreate it and existing data will be lost). Because you need a filesystem, you will end up using
one that include Journaling (afterall, they have disks in mind), which may decrease the performacne
compared to its `tmpfs` and `ramfs` counterpart, but it's the closest to emulating a real filesystem
as it can get. For instance, while you can't use `O_DIRECT` in `tmpfs`, you will be able to use it
on a `ramdisk`.

## Comparison table

| Feature | tmpfs | ramfs | ramdisk |
| ---------- | ------- | ------- | ---------- |
| Is a filesystem | Yes | Yes | No |
| Is a block device | No | No | Yes |
| Swaps | Yes | Yes* | No |
| Reserve unused RAM | No | No | Yes |
| Dynamically resizeable | Yes | Yes | No |
| Configurable | Yes | No | Yes |

[See official docs](https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html) for more information.

## Why use it at all?

Does caring too much about having persistent storage on memory matter? In most cases, not really.
[See this
answer](https://superuser.com/questions/1551982/should-i-expect-programs-that-run-from-a-tmpfs-folder-to-run-faster-with-and-w)
for more information, but summarizing:

> Running on tmpfs will be faster only if you have a lot of disk I/O that isn't fulfilled from page
> cache.
> 
> If the I/O is reads and the files are already in cache, tmpfs will make no difference.
> 
> If the I/O is async writes without flushing and the workload is bursty rather than continuous and
> there is enough cache to soak up a burst of writes and the writes can be flushed in the background
> between the bursts, tmpfs will make no difference.
> 
> If there is no disk I/O to be done, tmpfs will make no difference.
> 
> If you have a lot of disk I/O and your process is blocked iowaiting for storage to catch up,
> running on tmpfs will make a huge difference. The slower your disks, the more difference it will
> make, right up to the point where you become CPU bottlenecked.

All in all, most of the time you just make sure that you have enough memory and you will be fine, no
need for manual tuning such as this.

## Using ramdisk

```bash
modprobe brd rd_nr=1 rd_size=4585760 max_part=1
dd if=/dev/zero of=/dev/ram0 bs=300M count=1
sudo mkfs -t xfs -d size=300m,name=/dev/ram0

# Mounting it
mount -t xfs /dev/ram0 /mnt/path

# Removing /dev/ram0
umount /mnt/path
rmmod brd
```

## Using ramfs

Mind you that `ramfs` doesn't have any configurations to set:

```bash
mount -t ramfs none /mnt/path
```

## Using tmpfs

You can also use `noswap` as an option if you desire.

```bash
mount -t tmpfs -o size=1g[,noswap] none /mnt/path
```

### Resizing an existing mount

```bash
mount -o resize,size=2g /mnt/path
```
