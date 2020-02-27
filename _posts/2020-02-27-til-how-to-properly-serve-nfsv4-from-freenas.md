---
layout: post
title: 'TIL: How to properly serve NFSv4 from FreeNAS'
date: 2020-02-27 02:26:00 BRT
tags: linux, nfs, til
categories: articles
---

I was trying to serve a FreeNAS mount as a NFSv4 share. Unfortnately, there was one thing bugging me, which was this one:

```
> ls -la
total 1
-rw-r--r-- 1 nobody nobody 0 Feb 27 02:01 test
```

Yes, on the client, all files were showing up as being owned by `nobody`. After lots of debugging and wasted time, I found out that NFSv4 permission models requires the client and server to be at the same domain, so I checked with:

```
> sudo nfsidmap -d
localdomain
```

Then I went to the FreeNAS server and:

```
> hostname -d
lan
```

Oh Lord. It was just a matter of [switching my client machine to the new domain](https://serverfault.com/a/511304) and done.

```
> sudo vi /etc/hostname /etc/hosts
> sudo hostname -F /etc/hostname
```

Special thanks to [Martin Cheatle](https://www.ixsystems.com/community/threads/nfs-shares-are-mounted-as-nobody.44736/post-316720) that helped me in that gotcha moment.
