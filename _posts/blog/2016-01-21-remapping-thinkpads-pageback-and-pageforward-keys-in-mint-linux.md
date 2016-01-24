---
layout: post
title: "Re-mapping a keyboard layout in Linux Mint"
modified:
categories: blog
excerpt: "Assigning more useful functions to the keys on a laptop keyboard"
tags: [linux, mint, keyboard]
image:
  feature:
date: "2016-01-16"
---

## Obnoxious keyboard layout

The **Page Back** and **Page Forward** keys on my Lenovo ThinkPad T420 laptop have been giving me a lot of hard time. The keys are sitting next to the arrows keys making it very easy to accidentally hit **Page Back** or **Page Forward** when navigating with the arrows in a browser window, eventually losing all the data on the current page.

## xmodmap comes to help

I wanted to disable the **Page Back** and **Page Forward** keys or re-map them to something more useful, and found some advices how to use `xmodmap` for this. A post ["Mapping unsupported keys with xmodmap"][1] written by [Loki](https://www.blogger.com/profile/15179032995691105618) turned to be very helpful in my task. I have [Linux Mint](http://www.linuxmint.com/) installed on my machine, though this hack is applicable in any Linux distribution that has `xmodmap`.

## Reading out the key codes

To begin with, we need to know what codes send the keys we want to re-map. I will start the `xev` utility in a terminal and press the page-back and page-forward keys. Here is the command line:

```bash
xev | grep -A2 --line-buffered '^KeyRelease' | sed -n '/keycode /s/^.*keycode \([0-9]*\).* (.*, \(.*\)).*$/\1 \2/p'
```
It gives us the key codes in the first position and the key symbols in the second position:

    166 XF86Back
    167 XF86Forward

We also need to know key symbols to which we want to remap the keys. We want to remap the **Page Back** to **Page Up** and **Page Forward** to **Page Down**. While still having the `xev` command running, I hit the **Page Up** and **Page Down** keys and get their mappings:

    112 Prior
    117 Next

## Re-mapping the keys

Now, we want to remap **166** to **Prior** and **167** to **Next**. That will require overriding the default `xmodmap` configuration by a local `~/.Xmodmap` file. First, we'll get the original mappings by running the [xmodmap][2] command:
```bash
xmodmap -pke | grep "XF86Back\|XF86Forward" >> ~/.Xmodmap
```
Then the following default mapping lines captured in `~/.Xmodmap`

    keycode 166 = XF86Back NoSymbol XF86Back NoSymbol XF86Back XF86Back
    keycode 167 = XF86Forward NoSymbol XF86Forward NoSymbol XF86Forward XF86Forward

have to be edited, replacing the key symbols by the desired ones:

    keycode 166 = Prior NoSymbol Prior NoSymbol Prior Prior
    keycode 167 = Next NoSymbol Next NoSymbol Next Next

The local mappings file can be immediately activated by feeding it to `xmodmap`:
```bash
xmodmap ~/.Xmodmap
```

## Loading the mapping at startup

This is a bit tricky. The correct place to put `xmodmap ~/.Xmodmap` is in `~/.xinitrc`, as mentioned [here](http://askubuntu.com/questions/54157/how-do-i-set-xmodmap-on-login). Unfortunately, this way does not work in *Linux Mint Cinnamon Edition*, because the mapping is overridden by the desktop environment at startup. The solution is to add an command `/usr/bin/xmodmap /home/$USER/.Xmodmap` in *Startup Applications* with a delay of 15-20 seconds.

## Links

1. [Mapping unsupported keys with xmodmap][1]
2. [XMODMAP(1) manual page][2]

[1]: http://dev-loki.blogspot.se/2006/04/mapping-unsupported-keys-with-xmodmap.html
[2]: http://www.x.org/archive/X11R6.8.2/doc/xmodmap.1.html
