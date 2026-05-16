---
layout: post
title: "Debugging Firefox's 8-second startup delay on KDE Wayland"
date: 2026-05-16 00:00:00 -0300
tags: linux, debugging, firefox, kde
---

Every time I opened Firefox, there was an awkward 8-to-10-second wait before anything appeared on
screen. No splash, no window, just a spinning cursor and blind faith that something was happening.
It was consistent, reproducible with a brand-new profile, and annoying enough to be worth
investigating properly.

This is the story of that investigation.

## First steps: the obvious culprits

My first instinct was to look at things known to cause slow GTK application startups on KDE:
XDG Desktop Portals, the accessibility bus (AT-SPI), and GDK's Wayland backend initialization. I
tried disabling each in turn with environment variables — `GTK_USE_PORTAL=0`, `NO_AT_BRIDGE=1`,
`GTK_A11Y=none`, even `GDK_BACKEND=x11` to force X11 mode. None of them made any difference.
Firefox `about:config`'s own `widget.use-xdg-desktop-portal` toggle also had no effect.

## Getting serious: perf and strace

I ran `perf record` during startup and spotted `g_dbus_proxy_new_for_bus_sync` — a synchronous
D-Bus proxy creation — in the call stack. That was a tempting lead. But capturing the session bus
with `busctl monitor` showed only a brief flurry of `dconf` subscriptions right at startup, then
8 seconds of complete silence. Firefox wasn't blocking on D-Bus.

Time to use strace properly. I captured a trace with network and IPC syscalls:

```sh
strace -f -s 256 -e trace=network,ipc -o /tmp/ff.log firefox --new-instance --no-remote -P Test
```

This showed the crash reporter thread blocking on `poll([{fd=11}], 1, -1)` — an internal Firefox
socketpair — but I couldn't see *why* the main thread was so slow to send the signal that would
unblock it. The 8-second gap was invisible because `poll()` wasn't in my filter. I'd captured `ipc`
(which includes socketpairs) but missed the polling calls entirely.

I added `poll` to the filter and re-ran. This time I could measure it: the crash reporter's poll
took exactly 8.004 seconds. But the main thread still appeared to be doing fast individual operations
— ELF reads, ICE authentication, config file reads — each completing in microseconds.

The second mistake was subtler: my filter used `trace=poll`, which on modern Linux captures the
`poll` syscall, but **not** `ppoll` — they're separate syscalls. The main thread was using `ppoll`.

## Finding the real block

For the third strace run I switched to capturing blocking-oriented syscalls with absolute timestamps:

```sh
strace -f -tt -e trace=poll,ppoll,epoll_wait,futex,select -e signal=none -o /tmp/ff4.log \
  firefox --new-instance --no-remote -P Test
```

This immediately showed what had been invisible before. The main thread blocked in:

```
ppoll([{fd=20, events=POLLIN}], 1, {tv_sec=119, tv_nsec=999992000}, NULL, 8)
```

...and sat there for **3.95 seconds** before returning. Then, 4 seconds later, after a stretch
with no visible blocking calls, the crash reporter poll finally unblocked. Two separate ~4-second
delays, adding up to 8 seconds.

## Identifying the file descriptor

`ppoll` told me the *duration* but not the *destination*. I needed to know what fd=20 was. A quick
strace pass capturing only `connect` calls revealed it:

```sh
strace -f -tt -e trace=connect -e signal=none -o /tmp/ff5.log \
  firefox --new-instance --no-remote -P Test
```

```
connect(20, {sa_family=AF_UNIX, sun_path="/run/systemd/resolve/io.systemd.Resolve"}, 42) = 0
# ...3.8 seconds later...
connect(20, {sa_family=AF_UNIX, sun_path="/tmp/.ICE-unix/1242"}, 21) = 0
```

fd=20 was used first for a systemd-resolved query, then immediately reused for the ICE socket
connection. Something was being looked up in systemd-resolved and it was taking nearly 4 seconds
to respond.

## The actual query

systemd-resolved speaks the [varlink](https://varlink.org/) protocol. I needed to capture
`sendto`/`recvfrom` to see what was being asked:

```sh
strace -f -tt -s 256 -e trace=connect,sendto,recvfrom -e signal=none -o /tmp/ff7.log \
  firefox --new-instance --no-remote -P Test
```

```
sendto(20, "{\"method\":\"io.systemd.Resolve.ResolveHostname\",\"parameters\":{\"name\":\"archbox\",\"family\":2,\"flags\":0,\"ifindex\":0}}\0", ...)
recvfrom(20, 0x..., 131072, MSG_DONTWAIT, NULL, NULL) = -1 EAGAIN
# ... 3.8 seconds of retries ...
recvfrom(20, "{\"error\":\"io.systemd.Resolve.DNSError\",\"parameters\":{\"rcode\":3}}\0", ...)
```

Firefox was asking systemd-resolved to resolve `archbox` — the machine's short hostname — and
getting NXDOMAIN back after **3.8 seconds**. This happened **twice** in sequence, for a total of ~8
seconds.

## Why does Firefox look up its own hostname?

Firefox uses **libICE** (X Inter-Client Exchange) to register with the KDE session manager
(`ksmserver`). This is the legacy X11 session management protocol that allows a session manager to
save and restore application state across logins. When Firefox connects to ksmserver, ICElib
authenticates using MIT-MAGIC-COOKIE-1 tokens stored in `~/.ICEauthority`. To look up the right
token, it calls `getaddrinfo("archbox")` — the short hostname from `gethostname()`. It does this
**twice**: once for the ICE protocol layer, once for the XSMP sub-protocol.

## Why was the lookup so slow?

My machine's short hostname is `archbox`, and its FQDN is `archbox.home.lab`. The `/etc/hosts` file
already had:

```
127.0.0.1 archbox.home.lab
```

But **not** the bare `archbox`. When systemd-resolved was asked to resolve `archbox`, it had no
local match and fell back to LLMNR (Link-Local Multicast Name Resolution), broadcasting queries on
the local network and waiting for a response that never came — retrying several times before finally
giving up with NXDOMAIN.

## Finally, the fix

One line in `/etc/hosts`:

```
127.0.0.1 archbox
```

systemd-resolved checks `/etc/hosts` before attempting network queries. With that entry in place,
it returns immediately and Firefox starts in under a second.

This investigation was done with help from [Claude](https://claude.ai). I went in skeptical that an
AI would be genuinely useful for this kind of low-level systems debugging — the sort of work that
requires iterating on hunches, reading raw strace output, and knowing which tool to reach for next.
It turned out to be a good surprise. It didn't just suggest things to try; it reasoned through the
evidence with me and caught things I would have taken longer to notice on my own.

Fortunately the solution was simple, but I just wish this was easier to debug to be honest.
