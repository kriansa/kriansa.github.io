---
layout: post
title: How to make systemd units depend on specific hardware
date: 2024-12-27 12:00:00 BRT
tags: linux, systemd, udev
categories: articles
---

Sometimes you need a systemd service to wait for a specific hardware device to be available before
starting. For instance, when starting an X11 session as a service, you want to ensure the GPU is
ready.

By default, systemd only creates device units for block and network devices. Other devices, like
`/dev/dri/card0`, aren't exposed as units you can depend on.

The solution involves two steps: first, tell udev to tag the device for systemd, then use that
device as a dependency in your service.

## Creating a udev rule

Create a udev rule that tags the device with `systemd`:

#### /etc/udev/rules.d/99-dev-dri-card0.rules

```conf
ENV{DEVNAME}=="/dev/dri/card0", TAG+="systemd"
```

The `TAG+="systemd"` is what makes systemd [dynamically create][device-units] a device unit for this
device. Without it, systemd won't know about the device.

After creating the file, reload udev rules:

```sh
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## Using the device as a dependency

Now you can reference `dev-dri-card0.device` in your service units. The device path gets converted
to a unit name by replacing `/` with `-` and removing the leading slash.

#### /etc/systemd/system/xorg@.service

```conf
[Unit]
Description=X11 session for %i
Wants=dev-dri-card0.device
After=dev-dri-card0.device graphical.target systemd-user-sessions.service

[Service]
# ...
```

Using `Wants=` ensures the device is requested but won't fail the service if unavailable. The
`After=` directive ensures proper ordering—the service only starts after the device is ready.

Reload systemd to apply the changes:

```sh
sudo systemctl daemon-reload
```

[device-units]: https://www.freedesktop.org/software/systemd/man/systemd.device.html
