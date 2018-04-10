---
layout: post
title: Using Gnome gsettings on GDM user
date: 2018-04-10 01:30:00 BRT
tags: gnome, gdm, gsettings
categories: articles
---
[Arch GDM wiki](https://wiki.archlinux.org/index.php/GDM) suggests that if you
need to change some config for `gdm`, that must be done on `gdm` user. As a
first option, it's suggested that you run the command below to get shell access
to `gdm`:

```
# machinectl shell gdm@
```

That's unfortunally is not working:

```
Connected to the local host. Press ^] three times within 1s to exit session.
/sbin/nologin: invalid option -- 'l'
Try 'nologin --help' for more information.
Connection to the local host terminated.terminated
```

A [bug has been filed](https://github.com/systemd/systemd/issues/8634) on
upstream already, so that should be a matter of time to get it fixed on next
`systemd` release.

An alternative for that is running `su` enforcing a different shell:

```
# su gdm -s /bin/sh -c 'gsettings set org.gnome.desktop.interface scaling-factor 2' 
```

However, it fails me:

> (process:10904): dconf-WARNING **: 04:08:52.321: failed to commit changes to
> dconf: Cannot autolaunch D-Bus without X11 $DISPLAY

After [some googling](https://ubuntu-mate.community/t/error-cannot-autolaunch-d-bus-without-x11-display/11928),
I found a fix to that issue. We basically need to ensure that `dbus` is running
by prepending the command with
[dbus-launch](https://dbus.freedesktop.org/doc/dbus-launch.1.html).

```
su gdm -s /bin/sh -c 'dbus-launch gsettings set org.gnome.desktop.interface scaling-factor 2'
```

This should do the trick.
