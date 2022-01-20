---
layout: post
title: Using systemd as cron
date: 2022-01-13 23:00:00 BRT
tags: linux, systemd, cron
categories: articles
---

Tradionally, \*nix users are familiar _(or maybe not so much)_ with `cron` syntax, whereas you
define applications to be run at specific schedules.

What most people don't know is that you can also use the power of **Systemd** to do the same, and in
return get all the [benefits] by doing so, such as centralized logging through `journald`.

In order to use it you will need at least two files, one for a "service" unit, and another called
timer unit.

Here are the files and the contents of a minimal setup:

#### /etc/systemd/system/myprogram.service

```conf
[Unit]
Description=Runs my program which does something

[Service]
Type=oneshot
User=git
Group=git
ExecStart=/usr/local/bin/my-program
```

#### /etc/systemd/system/myprogram.timer

```conf
[Unit]
Description=Automatically runs myprogram every 15 minutes

[Timer]
OnCalendar=*:0/15

[Install]
WantedBy=timers.target
```

After you place both files at the specific location, you will need to reload systemd, then enable
and start the timer.

1. Reload systemd
   ```sh
   systemctl daemon-reload
   ```
2. Now enable and start the timer
   ```sh
   systemctl enable --now myprogram.timer
   ```

> Get familiar with the `OnCalendar` syntax on the systemd.timer(5) [manpage]. Also check out the
> incredible [Arch Guide] on the same subject.

[benefits]: https://wiki.archlinux.org/title/Systemd/Timers#As_a_cron_replacement
[manpage]: https://man.archlinux.org/man/systemd.timer.5
[Arch Guide]: https://wiki.archlinux.org/title/Systemd/Timers

## Automating it

For the best experience you should automate this. Here is the Ansible snippet to do so. Remember to
put both `.service` and `.timer` files under the `files` folder in the same level as this playbook
below.

```yaml
- hosts: all
  tasks:
    - name: create git-sync crontab entry
      become: true
      notify: 
        - reload systemd daemon
        - enable myprogram timer
      ansible.builtin.copy:
        src: "\{\{ item \}\}"
        dest: /etc/systemd/system/\{\{ item \}\}
      loop:
        - myprogram.service
        - myprogram.timer

  handlers:
    - name: reload systemd daemon
      become: true
      ansible.builtin.systemd: daemon_reload=yes

    - name: enable myprogram timer
      become: true
      ansible.builtin.systemd: name=myprogram.timer enabled=yes state=restarted
```
