---
layout: post
title: wmcompanion - Desktop environment features to your minimalist WM
date: 2022-10-20 23:00:00 BRT
tags: linux, tiling-wm
categories: projects
permalink: /projects/wmcompanion
github: kriansa/wmcompanion
---

You use a minimalist tiling window manager, yet you want to be able to tinker with your desktop more
easily and implement features like the ones available in full blown desktop environments?

More specifically, you want to react to system events (such as returning from sleep, or wifi signal
change) and easily automate your workflow or empower your desktop user experience using a consistent
and centralized configuration so it is actually easy to maintain?

**wmcompanion** is an automation tool that helps connecting system events to user-scriptable actions
in Python so you can easily implement more advanced features to your your minimalist tiling window
manager based desktop.

See a snippet of configuration code that maybe speaks for itself:

```python
@on(BluetoothRadioStatus)
@use(Polybar)
async def bluetooth_status(status: dict, polybar: Polybar):
    """
    Show the bluetooth status icon on Polybar
    This requires you to setup a polybar module using `custom/ipc` as the type
    """

    icon_color = "#F2F5EA" if status["enabled"] else "#999999"
    await polybar(module="bluetooth", polybar.fmt("[bt]", color=icon_color))
```

Use it to augment your existing status bar capabilities, properly react to system-level events such
as power and device connections, as well as give you more flexibility to customize your desktop in a
scriptable fashion, by centralizing your customizations and allowing you to share & reuse existing
configuration from others.

You can compare wmcompanion to [Hammerspoon][hammerspoon]{:target="_blank"} to a certain extent, but
it lacks the maturity and feature set of the latter. Also, if not obvious yet, wmcompanion is made
for GNU/Linux - although porting it to BSDs wouldn't be hard.

As of now, you can use it to react to events such as the following:

  * Audio input/output levels
  * Bluetooth state
  * Kbdd keyboard layout
  * NetworkManager connection status
  * NetworkManager Wi-Fi status
  * Dunst notification paused state
  * Power events such as returning from sleep, power source and battery levels
  * X11 display state
  * X11 input device (mice/keyboard) state

Given its flexibility, anyone can write and share new event listeners to it very easily, allowing
for high extensibility and reusability.

I advise you to take a look at the [README][README]{:target="_blank"} and the
[examples][examples]{:target="_blank"} so you can get inspired by what you can do.

[README]: https://github.com/kriansa/wmcompanion/blob/main/README.md
[examples]: https://github.com/kriansa/wmcompanion/tree/main/examples
[hammerspoon]: https://www.hammerspoon.org/
