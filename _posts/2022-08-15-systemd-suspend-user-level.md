---
layout: post
title: Using systemd user units to react to sleep/suspend
date: 2022-08-14 23:00:00 BRT
tags: linux, systemd
categories: articles
---

Using `systemd` to manage user-level services is great. There's a few caveats though, and one of
them is that there's no easy way to depend on system level units. 

That's because `systemd --user` runs as a separate process from the systemd --system process. User
units can not reference or depend on system units or units of other users. [[1]][arch-systemd-user]

While arguably safer, we could still benefit from the convenience that some units might bring to the
table, such as `sleep.target` and `suspend.target`. Those are the easiest way to run a service
before and after the computer slept.

There's however, a clean way to proxy out these hooks from the **system** to the **user** instance.
To do that, you will need to create a system unit that will trigger a user target:

#### /etc/systemd/system/suspend@.service

```conf
[Unit]
Description=Call user's suspend target after system suspend
After=suspend.target

[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl --user --machine=%i@ start --wait suspend.target

[Install]
WantedBy=suspend.target
```

Enable it by running `sudo systemctl daemon-reload && sudo systemctl enable suspend@$(whoami)`

Now we'll create a new user target that activates all needed user services:

#### ~/.config/systemd/user/suspend.target

```conf
[Unit]
Description=User level suspend target
StopWhenUnneeded=yes
Wants=alert-me.service
Wants=send-me-some-email.service
```

Notice that there's already a few example service specification as dependency on `Wants`. You can
declare as many as you want, as long as they're user units.

Edit the services you need and make sure you run `systemctl --user daemon-reload` after.

You will need to duplicate these files if you want to install it for `sleep` as well. But that's it,
from now on you are able to run user services on system sleep/suspend.

[arch-systemd-user]: https://wiki.archlinux.org/title/Systemd/User
