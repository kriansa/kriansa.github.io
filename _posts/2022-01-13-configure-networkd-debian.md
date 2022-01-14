---
layout: post
title: Configuring systemd-networkd on Debian
date: 2022-01-13 22:00:00 BRT
tags: linux, debian, systemd, systemd-networkd, network
categories: articles
---

By default, Debian uses `/etc/network/interfaces` file to manage your network interfaces by means of
`ifup` and `ifdown` commands.

If you're not happy with it, you can choose to use `NetworkManager` or systemd `networkd`. I'm using
the latter since it makes managing the interfaces very easily as I'm used to systemd.

If you simply want to have the interface configured as a DHCP client, this is what you need to do:

1. Backup your existing `/etc/network/interfaces`
   ```sh
   mv /etc/network/interfaces /etc/network/interfaces.save
   ```
2. Create a network configuration for systemd-networkd by placing the following content to the file
   `/etc/systemd/network/dhcp.network`:
   ```conf
   [Match]
   Name=en*
   
   [Network]
   DHCP=yes
   ```
3. Then you need to enable `systemd-networkd`
   ```sh
   systemctl enable systemd-networkd
   ```
4. Because at this point your network is probably working, the recommendation is that you reboot the
   computer so it can pick up the configuration by `systemd-networkd`.

For more complex setups, check out the official [docs].

## Automating it

Now, of course this is not optimal if manual, so here is the snippet if you need to automate it
with Ansible:

```yaml
- hosts: all
  tasks:
    - name: remove /etc/network/interfaces file
      become: true
      ansible.builtin.file:
        path: /etc/network/interfaces
        state: absent

    - name: configure the network using systemd-networkd
      become: true
      notify:
        - enable systemd-networkd
        - reboot
      ansible.builtin.copy:
        content: |
          [Match]
          Name=en*
          
          [Network]
          DHCP=yes
        dest: /etc/systemd/network/dhcp.network

  handlers:
    - name: enable systemd-networkd
      become: true
      ansible.builtin.systemd: name=systemd-networkd enabled=yes

    - name: reboot
      become: true
      ansible.builtin.reboot:
```

[docs]: https://wiki.debian.org/SystemdNetworkd
