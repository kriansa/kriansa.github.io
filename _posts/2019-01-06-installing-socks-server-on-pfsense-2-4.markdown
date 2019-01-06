---
layout: post
title: Installing a SOCKS server on pfSense 2.4+
date: 2019-01-06 13:00:00 BRT
tags: firewall, socks, pfsense, freebsd
categories: articles
---
Recently I needed to setup a SOCKS server on my LAN so that I could browser on
Firefox through proxy locally.

IMHO, it should be something built-in on pfSense, or at least very easy to
setup using the GUI. However, it requires many manual steps, which we're going
to cover now.

## Installing packages

To setup this server, we will need to install new packages, because there's
nothing built-in that you can achieve that.

[Dante](https://www.inet.no/dante/) is a free SOCKS server that we will use for
this server. We can find binary packages on FreeBSD repository, but
unfortunately pfSense does not use FreeBSD `pkg` repositories since 2.3.

We could enable FreeBSD repositories, but then many package dependencies would
conflict with those present on pfSense repos, so stay away from it.

There has been [some
discussion](https://forum.netgate.com/topic/97731/freebsd-packages-on-2-3rc)
around this subject, but long story short, the safest way to download a package
from the FreeBSD repository is by:

```
$ pkg add http://pkg.freebsd.org/FreeBSD:11:amd64/latest/All/<package>.txz
```

The only downside with this alternative is managing the package dependencies
and having to check for updates manually, something you wouldn't expect doing
when using a package manager.

## Accessing pfSense SSH

Before you can proceed, ensure that you have SSH access to your pfSense box.
That can be done by uploading your public key on the interface. Then while
connecting to it, remember that the username is the same that you use for
logging into the GUI interface.

## Install Dante

Dante can be installed by issuing the following commands:

```
# pkg install cyrus-sasl
# pkg add http://pkg.freebsd.org/FreeBSD:11:amd64/latest/All/miniupnpc-2.1_1.txz
# pkg add http://pkg.freebsd.org/FreeBSD:11:amd64/latest/All/dante-1.4.2_1.txz
```

Luckily, there's one dependency available on pfSense repos, so we don't need to
install it from FreeBSD (one less package to care about when doing upgrades).

### Configuring

Now you will need to setup the service. From my own requirements, I only needed
that it could listen into the right interface (`bridge0`) and redirect traffic
through the existing links (`pppoe0` and `pppoe1`). Also, I don't need
authentication, which is fine since we are serving only to LAN clients who
already have access to the whole outgoing network.

With those requirements in mind, here's the content that you should put into
your `/usr/local/etc/sockd.conf` (remember to edit it as `root`):

```
# Logging
logoutput: /var/log/sockd.log

# User
user.unprivileged: nobody

# Bind ports
internal: bridge0 port = 5000
external: pppoe0
external: pppoe1 
external.rotation: route

# Auth
clientmethod: none
socksmethod: none

client pass {
  from: 0.0.0.0/0 to: 0.0.0.0/0
  log: error
  clientmethod: none
}

# generic pass statement - bind/outgoing traffic
socks pass {
  from: 0.0.0.0/0 to: 0.0.0.0/0
  command: bind connect udpassociate bindreply udpreply
  socksmethod: none
  log: error
}
```

This is enough, now you can start the service:

```
# /usr/local/etc/rc.d/sockd onestart
```

### Enabling the service on startup

You will want to have this running when your box restarts. You will need to add
it to the configuration file (`config.xml`).

Edit the file `/cf/conf/config.xml`, and look for a specific section where it's
closing the `</system>` tag.

Then add the following snippet:

```xml
<shellcmd>/usr/local/etc/rc.d/sockd onerestart</shellcmd>
```

That should do it.

## Maintenance routine

Well, because we're kinda off the grid here, we will need to do some manual
work. First of all, we just installed two packages that are not on the machine
repo. Every time they need to update, we will need to do it manually, so take
care of it, you don't want to have a router software with vulnerabilities on
your network.

Take a look at [Dante](https://www.inet.no/dante/) and
[MiniUPnP](http://miniupnp.free.fr/files/) websites every now and then
to see when there are new releases. Also, take a look at the CVE boards and
security advisories.

## Links

* [Dante Docs 1.4.x](https://www.inet.no/dante/doc/1.4.x/index.html)
* [FreeBSD 11 Package Repository](http://pkg.freebsd.org/FreeBSD:11:amd64/latest/All/)
* [Installing Dante on pfSense](https://forum.netgate.com/topic/139300/socks5-proxy-dante-on-virtual-ip-to-use-openvpn-ovpnc1-as-gateway)
* [Install FreeBSD Packages on pfSense](https://forum.netgate.com/topic/97731/freebsd-packages-on-2-3rc)
* [Enable SOCKS Proxy on pfSense](https://forum.netgate.com/topic/81228/socks5-proxy/12)
