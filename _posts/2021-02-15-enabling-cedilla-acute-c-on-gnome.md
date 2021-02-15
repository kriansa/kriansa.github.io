---
layout: post
title: Enabling C cedilla (ç) on GNOME - updated way
date: 2021-02-15 16:10:00 BRT
tags: linux, gnome, keyboard
categories: articles
---

I use an english keyboard but also happen to type in Portuguese. To type in accents, a standard way
is adding a new keyboard source (English alt. intl.) and use dead-keys to compose accents. This
works pretty well in most cases, except many people (like me) are used to type `' + c` to get a `Ç`
(C cedilla), but instead, with the default configs, an accented `Ć` is what you get.

There are several ways of solving this issue and they changed over time. Nevertheless, when you
google for it, you'll find solutions that includes modifying system files such as
`/usr/lib/gtk-3.0/3.0.0/immodules.cache` or `/usr/share/X11/locale/en_US.UTF-8/Compose` and
`/etc/environment`. While these solutions may work, I was trying fix it within my own dotfiles and
not touching system files if possible.

After some digging, here is the final solution, and it works for both GTK and non-GTK applications.

1. Create a `~/.XCompose` file with the following content:

    ```
    # UTF-8 (Unicode) compose sequences

    # Overrides C acute with Ccedilla:
    <dead_acute> <C> : "Ç" "Ccedilla"
    <dead_acute> <c> : "ç" "ccedilla"
    ```

2. Run the following command:

    ```sh
    $ gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "{'Gtk/IMModule': <'ibus'>}"
    ```

3. Reboot or restart X11

What it does is basically add a `XCompose` override at your home folder. GTK also use it as a source
for [its own compose][gtk-compose] table. However, it won't just work unless you tell it to use
`ibus` as the input method. One way of doing so would be just setting this property:
`org.gnome.desktop.interface` `gtk-im-module`, but unfortunately any changes you do at this property
[will be reset][gsd-kbd-reset] when you reload the GNOME session. That's because
`gnome-settings-daemon` will define this value based on whether or not the current input source
needs `ibus` or not. Luckily, there's a [way of overriding][gsd-overrides] that value - setting
`org.gnome.settings-daemon.plugins.xsettings` `overrides` values.

Hopefully it is just enough to work fine on modern systems (GNOME 3.38+), but let me know if this
doesn't work for you.

[gtk-compose]: https://gitlab.gnome.org/GNOME/gtk/blob/2f43b8dc49491c1dd73248326722eeb12029d95f/gtk/gtkcomposetable.c#L76
[gsd-kbd-reset]: https://gitlab.gnome.org/GNOME/gnome-settings-daemon/-/blob/12388c35562acea42b3e4c52c85271c435151d63/plugins/keyboard/gsd-keyboard-manager.c#L303
[gsd-overrides]: https://gitlab.gnome.org/GNOME/gnome-settings-daemon/-/blob/5b065ee0bcc47d8e1c1eb0584b123427438b0512/plugins/xsettings/gsd-xsettings-manager.c#L481
