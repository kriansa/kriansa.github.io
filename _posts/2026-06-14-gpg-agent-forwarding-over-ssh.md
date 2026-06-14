---
layout: post
title: "Forwarding your GPG agent over SSH"
date: 2026-06-14 00:00:00 -0300
tags: gpg, ssh, gnupg, yubikey
---

If you keep your GPG keys on a smartcard (like a Yubikey as I do), you can use them on a remote host
over SSH without ever copying a private key off the card. The signing and decryption happen on your
local computer, driven by the remote; and the keys never leave the hardware. This is *GPG agent
forwarding*, and it's very useful for signing commits or decrypting files on a remote computer.

The problem is that almost every guide, including GnuPG's own example tells you to forward the
agent's *restricted* (called extra) socket. On recent GnuPG (2.4.x) that silently breaks signing and
decryption. Here's the setup that actually works.

## Forwarding locally

`gpg` is just a front-end; the secret-key operations live in `gpg-agent`, which talks to your card
over a Unix socket. Forward your local agent's socket onto the remote and the remote's `gpg` talks
to *your* agent (and *your* card). If configured, the PIN prompt (and any touch) happen locally.

Add this to your `~/.ssh/config`:

```
Host devbox
  # Host <yourhost>
  RemoteForward /run/user/<your remote UID>/gnupg/S.gpg-agent /run/user/<your local UID>/gnupg/S.gpg-agent
```

The order is `RemoteForward <remote-socket> <local-socket>`. Find the exact paths with `gpgconf
--list-dirs agent-socket` on each machine -- it's usually `/run/user/<uid>/gnupg/...`, and mind you
that the uid can differ between hosts.

## Use the full socket, not the "extra" one

This is the part people get wrong. `gpg-agent` listens on two sockets:

- `S.gpg-agent` — the full socket.
- `S.gpg-agent.extra` — a *restricted* socket that only permits a safe subset of commands. Every
  forwarding guide points you here, for a good reason: you don't want a remote box to be able to
  manage or wipe your keys.

The problem is that recent `gpg` discovers which secret keys exist by asking the agent `HAVEKEY
--list`, and the restricted socket answers `Forbidden`. `gpg` has no per-key fallback, so it decides
there's no secret key and refuses to work:

```
gpg: skipped "0xDEADBEEF": No secret key
gpg: signing failed: No secret key
```

To make it more confusing, *encryption still works* — that's a public-key operation that never
touches the agent. Only signing and decryption break. The fix is to forward the **full**
`S.gpg-agent`, exactly as shown above, and not `S.gpg-agent.extra`. There's a real trade-off to this;
see the security note at the end.

## On the remote: 

### 1. Let sshd replace a stale socket

By default `sshd` refuses to bind the forwarded socket if a file already exists at that path. Allow
it to clean up first, via `/etc/ssh/sshd_config` (or better, just add it to a file under
`/etc/ssh/sshd_config.d/`):

```
StreamLocalBindUnlink yes
```

Then `systemctl reload sshd`.

### 2. Import your public key

Forwarding carries only the *private* side. The remote's `gpg` still needs your **public** key to
know what it's signing with:

```sh
gpg --export <your-key-id> | ssh <yourhost> gpg --import
```

Don't bother testing with `gpg --card-status` — just sign or decrypt something
(`echo test | gpg --clearsign`). Your YubiKey should blink at home and the PIN prompt should appear
on your desktop.

### 3. Make the socket directory exist at boot

The first time you connect after the remote reboots, you may hit this:

```
Warning: remote port forwarding failed for listen path /run/user/<UID>/gnupg/S.gpg-agent
```

`sshd` binds the forwarded socket during session setup, but `/run/user/<uid>/gnupg` doesn't exist
yet — nothing has created it, and `sshd` won't `mkdir` a forward's parent. Normally that directory
is created lazily by the first `gpg` command, which is too late for this first connection.

Fix it with a user-level `tmpfiles` rule on the remote. Create `~/.config/user-tmpfiles.d/gnupg.conf`
with:

```
d %t/gnupg 0700 - - -
```

`%t` expands to `/run/user/<uid>`. 

That's it.

## A security note

The full socket gives the remote host unrestricted access to your *entire* agent for as long as
you're connected — every key it holds, not just the card — including key-management commands. And by
default GnuPG caches your card PIN for ten minutes, so during that window anything running as you on
the remote can sign or decrypt with no prompt at all. You're better be doing this to hosts you
trust.

One thing worth doing to increase the security is adding a touch requirement to the card, if
available. On a YubiKey, `gpg --card-edit` → `admin` → `uif 1 on` (signing) and `uif 2 on`
(decryption) makes every operation need a physical tap - so even with the full socket and a cached
PIN, a compromised remote can't use your keys without you. Use `on`, not `permanent`.

With touch enabled, the trade-off becomes pretty comfortable: your keys stay on the card, they work
on the remote, and nothing happens without your finger on the YubiKey.
