---
layout: post
title: "Make Any Terminal Emulator a Quake Style Dropdown"
date: 2014-07-19 13:33:53 -0400
comments: true
published: true
categories: terminal tmux dropdown linux xdotool wmctrl
---
## A Window Manager Independent Solution
There are plenty of existing dropdown terminal emulators such as [guake](https://github.com/Guake/guake), [tilda](http://tilda.sourceforge.net/tildadoc.php), [xfce4-terminal](http://docs.xfce.org/apps/terminal/dropdown), [yakuake](http://yakuake.kde.org/), [urxvt](http://software.schmorp.de/pkg/rxvt-unicode.html) (with -pe quake), and finalterm. Like splits and tabs (which can be done with many programs regardless of whether the terminal has native support), I don't consider the ability to hide and show a term window in the same location to be an important factor in choosing a terminal. There are plenty of good terminals that do not have this feature, and there are plenty of solutions to add it. The problem I encountered is that none of these solutions are universal. Most scripts will only work with a few terminals ([yeahconsole](http://phrat.de/yeahtools.html)) or require a certain window manager (e.g. [awesome](http://awesome.naquadah.org/wiki/Drop-down_terminal)).

When I switched from urxvt to [termite](https://github.com/thestinger/termite), I found myself in want of a way to use termite as a dropdown. I was using primarily xmonad and awesome at the time, couldn't get the above awesome script working, and wasn't sure if I would stick with awesome anyway. I couldn't find any general scripts that worked, so I ended up making a simple one using a tmux session to save the terminal state instead of hiding a window using some specific window manager feature.

I'll list three methods below that can be modified to potentially work with most any terminal and window manager. The one I am currently using is the one that makes use of [xdotool](http://www.semicomplete.com/projects/xdotool/)'s windownunmap features as I find it to be by far the best and simplest.

## Using Tmux
I already almost always use tmux, so using it for this purpose is not a problem for me. For anyone not interested in using a multiplexer, see the solution below.

I won't include the script I used, because I'm now using the method below since it eliminates some of the problems listed above. It can be viewed [here](https://github.com/angelic-sedition/dotfiles/blob/master/not_in_use/bin/my_dropdown.sh). However, here's a video of what the result looks like:

{% youtube 3yhX_y1VdWE %}

Since tmux sessions persist even if you kill the terminal they're running in, a terminal window can be killed instead of hidden on a hotkey press. A subsequent hotkey press can open a new instance of the terminal emulator, position it properly, and then reattach to the tmux session. This can be done by binding a key to run a script using a hotkey program like [shxkd](https://github.com/baskerville/sxhkd) or xbindkeys. The script will check the class of the current window with xprop (or xtitle, which is often easier) and close it if it's your terminal emulator or open your terminal emulator and reattach to the session if it's not.

While this method is relatively simple, there are a few downsides. If you use non-dropdown terminals that are the same program (e.g. xterm) as your dropdown terminal, this method will kill them if they are focused instead of opening your dropdown (the method below improves somewhat on this behaviour). As for speed, I've found this method to sometimes have noticeable delay and be slower than an actual dropdown terminal like guake. This is usually only a problem where things are already slow due to heavy RAM usage, for example.

As for adapting this method to window managers with less scriptability than [bspwm](https://github.com/baskerville/bspwm) or [herbstluftwm](http://herbstluftwm.org/), it may be slightly more difficult. All actions for bspwm are shell commands, so closing a window in a shell script is as simple as `bspc window -c`. For many window managers, using [wmctrl](http://tomas.styblo.name/wmctrl/) (e.g. `wmctrl -c :ACTIVE:`) may work just fine. Otherwise, either killing the terminal or faking the close window shortcut (e.g. alt+f4) using something like xdotool may be necessary.

Even more difficult may be positioning the terminal window properly after opening it. For tiling window managers that don't support floating, there's not much that can be done. Window managers that do support floating usually have the option of always starting a certain program floating. Bspwm also allows for a oneshot rule to float a certain program the next time that it is opened: `bspc rule -a termite -o floating=on`. As for resizing and window movement, I'm using [xdo](https://github.com/baskerville/xdo). If this does not work, using wmctrl's '-e' flag may be another possibility. As a last resort, it should be possible to resize a window using xdotool's mousemove, mousedown, and mouseup (and possibly keyup and keydown; I've left an example in the script above, but this is a really ugly and slow way to do things).

## Using Xdotool
Thanks to [quiv and Supplantr](https://bbs.archlinux.org/viewtopic.php?pid=1436909#p1436909) on the arch forums, I found out about xdotool's `windowunmap` feature. Using this script, it is possible to "minimize" or hide a window easily instead of killing the terminal and reattaching to tmux.

```
#!/bin/bash

if [ -n "$1" ]; then
    xdotool search --onlyvisible --classname $1 windowunmap || xdotool search --classname $1 windowmap || exec $1 &
else
    echo "Error: no argument supplied. Aborting."
fi
```

Using quiv's script is as simple as running `chmod +x` on it and binding it to a hotkey in the `.sxhkdrc`:
```
alt + s
	/path/to/script.sh <your terminal; e.g. termite>
```

This eliminates some of the previously mentioned problems. For one, it's generally faster since it is not necessary to start a new window and reattach to the tmux session. Because the window is not killed, it is no longer necessary to resize it every time. With the tmux method, the resizing can be seen every time the terminal is opened if the user does not have fade in compositing. Also, this method gives a true toggle where whether the hotkey closes or opens the dropdown terminal does not depend on what window is focused. This is how guake and most other dropdowns behave. However, this behaviour can be changed so that, like the old script, it will hide the terminal if it isn't active or activate it if it isn't focused but isn't hidden either.

The best improvement for me is that this script does not cause ranger to crash upon toggling when an image is being previewed. The previous method will crash ranger or break image previewing.

This script as is does not solve the problem for those who uses non-dropdown windows of the same type. I don't find more than one termite window to be necessary with tmux (I'd rather use multiple sessions or a different dedicated dropdown), but for those who will have multiple terminal windows open of the type specified in the script, this not a viable solution unmodified. A workaround probably wouldn't be too hard to make by using window id or by giving the terminal a specific title when creating it. However, it should be noted that the same is true of normal dropdown terminals; as far as I am aware, you can't have multiple guake instances running for example. One thing to note is that if you're only using a terminal of the same type for a single task, this isn't really a problem. For example, I'm using a separate termite instance to choose the location of a file on saving (see my last post). I actually find this behaviour useful in this case. For example, if a download finishes and the termite instance pops up, but you're in the middle of typing something, you can use your dropdown key to hide it, finish what you're doing, then use the hotkey to bring it back up. Once you quit that terminal, the dropdown hotkey can be used for your actual dropdown again.

Another potential problem of this script is that if you hide and show your dropdown while another floating window is up, it may mess up the sizing of your dropdown terminal. I'm not sure if this is because of the window manager that I'm using, but If I open and close my dropdown with guake open, for example, the size of the terminal used with this script gets shrunk to a very small square (at which point I have to kill the window or toggle it a bunch of times to fix the problem). Using xdo for hiding and showing instead fixes this problem but introduces other problems. Xdo can't only act on visible windows, so xdotool has to be used to activate the window first (and if that's successful, it can then be hidden for a true toggle). Xdo also can't deal with multiple instances like xdotool can. Instead of switching to hiding and showing a new instance, xdo will hide and show both.

Using a combination of both xdotool and xdo can fix this though. To summarize, xdotool's hiding can mess up window size. Xdo's show (and hide, though this doesn't matter as much) will show all hidden windows of the type and will bring up an extra black window when used with termite. Xdotool's hide (windowunmap) sucks and xdo's show sucks. It's almost too perfect. The one problem that remains after combinding the good parts is that hiding a fullscreen window seems to always mess up the sizing regardless of how it is hidden.

For tmux users, the above script can also be modified to allow reattaching to a tmux session in the case that the window is don't closed. Adding initial resizing and tmux session setup to the script is simple. Here are two versions of the modified script. I've left the first just because it is what I used to use and may work fine when modified for a different window manager.

```
if [ "$1" == "termite" ]; then
	bspc rule -a termite -o floating=on
	xdotool search --onlyvisible --classname $1 windowunmap || xdotool search --classname $1 windowmap || termite -e "/bin/zsh -c 'sleep 0.01 && xdo move -x 0 -y 16 && xdo resize -w +800 && tmux attach-session -dt dropdown || tmuxinator dropdown'" &
else
	xdotool search --onlyvisible --classname $1 windowunmap || xdotool search --classname $1 windowmap || exec $1 &
fi
```
Note that if you're using bspwm, the oneshot floating rule can also be put in front of the opening parts to prevent a newly opened termite instance from floating after hiding the termite dropdown (see the next example). This will make it so it will only set up the floating rule before opening the dropdown.

Example using terminology (this is what I'm currently using):
```
if [ "$1" == "Terminology" ]; then
	(xdotool search --onlyvisible --classname main windowactivate && xdo hide -n main) ||\
	(bspc rule -a main -o floating=on && xdotool search --classname main windowmap) ||\
	(bspc rule -a main -o floating=on && terminology -f=Inconsolata/12 -G=false -2 -e "/bin/zsh -c 'sleep 0.01 && xdo move -x 0 -y 14 && xdo resize -w +880 && tmux attach-session -dt pretty || tmux new-session -s pretty'" &)
fi
```
It's probably not worth using tmux with terminology though since you lose all its cool features. This is a toggle. If you want it to activate the dropdown when it's not focused instead of hiding it, you can remove the windowunmap on the second line.

Anyone who doesn't use tmuxinator will want to replace the corresponding part with something along the lines of `tmux new-session -s dropdown`.

## Using Wmctrl
The first thing I found when looking for a solution was [quake_term.sh](https://github.com/jw013/quake_term/blob/master/quake_term.sh) which uses wmctrl for hiding. I was unable to get this script working with any window manager at the time. Also, this method is slightly more complicated than using xdotool for hiding and suffers from some of the problems that the xdotool method solves.

## Fullscreen Toggle
Often dropdown terminals will provide a configurable hotkey for putting the terminal into fullscreen. Bspwm users can simply use `bspc desktop -l next` for monocle toggle or `bspc window -t fullscreen` for actual fullscreen. I have these bound in specific programs, but they can also be bound universally with sxhkd. For those not using bspwm, it may be possible to bind a key to something like `wmctrl -r :ACTIVE: -b toggle,fullscreen` instead for the same functionality. Note that, like I previously mentioned, hiding a window that is fullscreen will mess up the sizing using the xdotool method.

## Closing Notes
Don't let someone tell you to use crap like terminator over a terminal emulator you like just for native tabs and splits and don't think it's necessary to abandon your favourite lightweight terminal in favour of another just for native dropdown support!
