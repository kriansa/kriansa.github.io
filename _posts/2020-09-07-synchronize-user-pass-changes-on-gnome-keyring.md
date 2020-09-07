---
layout: post
title: Synchronizing password changes on Gnome-Keyring
date: 2020-09-07 16:10:00 BRT
tags: linux, gnome, gnome-keyring, seahorse
categories: articles
---

I have a good habit of changing my system password every 3 months. However, recently I noticed that
when I changed it on GNOME interface, it didn't reflect on Gnome Keyring (which is the backend for
[Seahorse](https://wiki.gnome.org/Apps/Seahorse)).

When looking for why that was happening, I didn't find good answers. I was a bit disappointed with
some answers that suggested removing the whole keyring as a "solution". Anyway, I'm quite convinced
that this does not affect the majority of Ubuntu users, as this [bug][ubuntu-bug] points out this
was fixed some time ago.

_I use Arch BTW_, and when on GNOME, this bug seems to be affecting us. After some digging, I found
on [GNOME Manual][gnome-manual] a entry on how to fix that. It turns out that we need to let passwd
PAM integration know that it needs to call the GNOME-keyring extension in order to synchronize the
password change with the `login` keyring.

So in short, here's the solution. Add the following line to the `/etc/pam.d/passwd` file:

```
password	optional	pam_gnome_keyring.so
```

There's no need to restart or reload any daemon. Just doing that should do the work. Now to test it,
make sure you have the same password for your user and your `login` keyring and then trigger a
password change (either on terminal using `passwd` or using GNOME-settings interface).

Meanwhile, I opened a [bug][arch-bug] on Arch bugtracker and let's hope it gets fixed.

[ubuntu-bug]: https://bugs.launchpad.net/gnome-keyring/+bug/416825
[gnome-manual]: https://wiki.gnome.org/Projects/GnomeKeyring/Pam/Manual
[arch-bug]: https://bugs.archlinux.org/task/67846
