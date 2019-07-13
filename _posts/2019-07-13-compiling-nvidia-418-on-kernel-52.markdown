---
layout: post
title: Compiling NVIDIA 418.74 Driver on Kernel 5.2
date: 2019-07-13 14:21:00 BRT
tags: linux, nvidia
categories: articles
---
With the recent release of [Kernel 5.2](https://lkml.org/lkml/2019/7/7/281), compiling NVIDIA 418.74 drivers is now shouting this error:

```
/NVIDIA-Linux-x86_64-418.74/kernel/nvidia-uvm/uvm8_tools.c:209:13: error: conflicting types for ‘put_user_pages’
  209 | static void put_user_pages(struct page **pages, NvU64 page_count)
      |             ^~~~~~~~~~~~~~
In file included from /NVIDIA-Linux-x86_64-418.74/kernel/common/inc/nv-pgprot.h:17,
                 from /NVIDIA-Linux-x86_64-418.74/kernel/common/inc/nv-linux.h:20,
                 from /NVIDIA-Linux-x86_64-418.74/kernel/nvidia-uvm/uvm_linux.h:41,
                 from /NVIDIA-Linux-x86_64-418.74/kernel/nvidia-uvm/uvm_common.h:48,
                 from /NVIDIA-Linux-x86_64-418.74/kernel/nvidia-uvm/uvm8_tools.c:23:
./include/linux/mm.h:1075:6: note: previous declaration of ‘put_user_pages’ was here
 1075 | void put_user_pages(struct page **pages, unsigned long npages);
      |      ^~~~~~~~~~~~~~
make[2]: *** [scripts/Makefile.build:279: /NVIDIA-Linux-x86_64-418.74/kernel/nvidia-uvm/uvm8_tools.o] Error 1
```

It turns out that `put_user_page` is [now being provided][kernel-commit] by the
kernel and it conflicts with the existing one by NVIDIA drivers.

What one can do to ensure the drivers will compile just fine is to comment out
the functions present on `kernel/nvidia-uvm/uvm8_tools.c`. Below you'll find
this patch that you can add to your compile script.

```text
--- a/kernel/nvidia-uvm/uvm8_tools.c
+++ b/kernel/nvidia-uvm/uvm8_tools.c
@@ -204,12 +204,14 @@ static bool tracker_is_counter(uvm_tools_event_tracker_t *event_tracker)
     return event_tracker != NULL && !event_tracker->is_queue;
 }

+/*
 static void put_user_pages(struct page **pages, NvU64 page_count)
 {
     NvU64 i;
     for (i = 0; i < page_count; i++)
         put_page(pages[i]);
 }
+*/

 static void unmap_user_pages(struct page **pages, void *addr, NvU64 size)
 {
```

If you happen to use Arch Linux, I've added this to my own [nvidia-418
package][nvidia-aur].

## Credits

Well, I didn't find this on my own. Thanks [myn][myn-post] for sharing your
findings.

[kernel-commit]: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?id=fc1d8e7cca2daa18d2fe56b94874848adf89d7f5
[nvidia-aur]: https://github.com/kriansa/PKGBUILDs/tree/master/pkgs/nvidia-418
[myn-post]: https://myn.meganecco.org/1560323400.html
