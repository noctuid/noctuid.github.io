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

I'll list three methods below that can be modified to potentially work with most any terminal and window manager. The one I am currently using is the one that makes use of [xdotool](https://github.com/jordansissel/xdotool)'s windownunmap feature as I find it to be by far the best and simplest. My personal version of the script that works with bspwm can be found [here](https://github.com/angelic-sedition/dotfiles/blob/master/scripts/bin/hide_show). It can also be used with windows that are not terminals and provides options for sizing and initial placement.

## Using Tmux
I already almost always use tmux, so using it for this purpose is not a problem for me. For anyone not interested in using a multiplexer, see the solution below.

I won't include the script I used, because I'm now using the method below since it eliminates some of the problems listed above. It can be viewed [here](https://github.com/angelic-sedition/dotfiles/blob/master/not_in_use/bin/my_dropdown.sh). However, here's a video of what the result looks like:

{% youtube 3yhX_y1VdWE %}

Since tmux sessions persist even if you kill the terminal they're running in, a terminal window can be killed instead of hidden on a hotkey press. A subsequent hotkey press can open a new instance of the terminal emulator, position it properly, and then reattach to the tmux session. This can be done by binding a key to run a script using a hotkey program like [shxkd](https://github.com/baskerville/sxhkd) or xbindkeys. The script will check the class of the current window with xprop (or xtitle, which is often easier) and close it if it's your terminal emulator or open your terminal emulator and reattach to the session if it's not.

While this method is relatively simple, there are a few downsides. If you use non-dropdown terminals that are the same program (e.g. xterm) as your dropdown terminal, this method will kill them if they are focused instead of opening your dropdown (the method below improves fixes this). As for speed, I've found this method to sometimes have noticeable delay and be slower than an actual dropdown terminal like guake. This is usually only a problem where things are already slow due to heavy RAM usage, for example.

As for adapting this method to window managers with less scriptability than [bspwm](https://github.com/baskerville/bspwm) or [herbstluftwm](http://herbstluftwm.org/), it may be slightly more difficult. All actions for bspwm are shell commands, so closing a window in a shell script is as simple as `bspc window -c`. For many window managers, using [wmctrl](http://tomas.styblo.name/wmctrl/) (e.g. `wmctrl -c :ACTIVE:`) may work just fine. Otherwise, either killing the terminal or faking the close window shortcut (e.g. alt+f4) using something like xdotool may be necessary.

Even more difficult may be positioning the terminal window properly after opening it. For tiling window managers that don't support floating, there's not much that can be done. Window managers that do support floating usually have the option of always starting a certain program floating. Bspwm also allows for a oneshot rule to float a certain program the next time that it is opened: `bspc rule -a termite -o floating=on`. As for resizing and window movement, I'm using xdotool. It's nice because you can specify the X and Y as percentages. If this does not work, using xdo or wmctrl's '-e' flag may be other possibilities. As a last resort, it should be possible to resize a window using xdotool's mousemove, mousedown, and mouseup (and possibly keyup and keydown; I've left an example in the script above, but this is a really ugly and slow way to do things).

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

This eliminates some of the previously mentioned problems. For one, it's generally faster since it is not necessary to start a new window and reattach to the tmux session. Because the window is not killed, it is no longer necessary to resize it every time. Also, this method gives a true toggle where whether the hotkey closes or opens the dropdown terminal does not depend on what window is focused. This is how guake and most other dropdowns behave. However, this behaviour can be changed so that, like the old script, it will hide the terminal if it isn't active or activate it if it isn't focused but isn't hidden either.

The best improvement for me is that this script does not cause ranger to crash upon toggling when an image is being previewed. The previous method will crash ranger or break image previewing.

This script as is does not solve the problem for those who uses non-dropdown windows of the same type, but I currently use a heavily modified version that fixes this problem by using window id to prevent other windows of the same classname from being affected. I haven't, however, added the ability to have multiple dropdowns of the same type. This would be relatively simple to do by saving window id in a file by a name given to the dropdown instead of by the name of the program, but isn't something I think I would ever use. I've kept a generic toggle function that preserves this behaviour, as it can sometimes be useful. For example, I'm using a separate termite instance to choose the location of a file on saving (see my last post). If a download finishes, and the termite instance pops up, but you're in the middle of doing something, you can use your dropdown key to hide it, finish what you're doing, then use the hotkey to bring it back up. Once you quit that terminal, the dropdown hotkey can be used for your actual dropdown again.

Another potential problem of this script is that if you hide and show your dropdown while another floating window is up, it may mess up the sizing of your dropdown terminal. I'm not sure if this is because of the window manager that I'm using, but If I open and close my dropdown with guake open, for example, the size of the terminal used with this script gets shrunk to a very small square (at which point I have to kill the window or toggle it a bunch of times to fix the problem). Using xdo for hiding and showing instead fixes this problem but introduces other problems. Xdo can't only act on visible windows, so xdotool has to be used to activate the window first (and if that's successful, it can then be hidden for a true toggle). Xdo also can't deal with multiple instances like xdotool can. Instead of switching to hiding and showing a new instance, xdo will hide and show both.

Using a combination of both xdotool and xdo can fix this though. To summarize, xdotool's hiding can mess up window size. Xdo's show (and hide, though this doesn't matter as much) will show all hidden windows of the type and will bring up an extra black window when used with termite. Xdotool's hide (windowunmap) sucks and xdo's show sucks. It's almost too perfect. The one problem that remains after combining the good parts is that hiding a fullscreen window seems to always mess up the sizing regardless of how it is hidden.

For tmux users, the above script can also be modified to allow reattaching to a tmux session in the case that the window is don't closed. Adding initial resizing and tmux session setup to the script is simple.

Example generic toggle for bspwm using classname:
```
generic_term_toggle() {
	# first argument is terminal name; second is the name of the tmux session to be used
	# this is from a larger script so to be used standalone, the width, height, etc. parts should be replaced with desired values
	(xdotool search --onlyvisible --classname $1 windowactivate && xdo hide -n $1) ||\
	(bspc rule -a $1 -o floating=on && xdotool search --classname $1 windowmap) ||\
	(bspc rule -a $1 -o floating=on && $1 -e "/bin/bash -c 'sleep 0.01 && xdotool getactivewindow windowmove $xoff $yoff windowsize $width $height && tmux attach-session -dt $2 || tmuxinator start $2 || tmux new-session -s $2'")
}
```
The `/bin/bash -c` part isn't necessary for most terminals with the `-e` flag, but for some reason, termite will give an error along the lines of `sleep invalid option 'd'` with just `-e "sleep..."`.

This is a toggle, but it's fairly easy to change it to do something like activate the window if it is visible but not active and only hide when the window is active. For the actual script that uses window id, see [here](https://github.com/angelic-sedition/dotfiles/blob/master/scripts/bin/hide_show).

## Using Wmctrl
The first thing I found when looking for a solution was [quake_term.sh](https://github.com/jw013/quake_term/blob/master/quake_term.sh) which uses wmctrl for hiding. I was unable to get this script working with any window manager at the time. Also, this method is slightly more complicated than using xdotool for hiding and suffers from some of the problems that the xdotool method solves.

There is a [much better script](https://bbs.archlinux.org/viewtopic.php?pid=1484774#p1484774) by lharding that uses wmctrl. It uses his [xtoolwait fork](https://github.com/lharding/xtoolwait) to get the window id, meaning that this can be used for windows of any program without interfering with other windows that share the same classname or even pid. I've integrated this sort of behaviour into my script. However, with bspwm at least, not all of wmctrl's features work (for example toggling a window's hidden state). I personally much prefer xdotool over wmctrl.

## Extras
**_Opening Windows From Dropdown_**

I only really open windows from within ranger (using rifle). It can be annoying to open sxiv or mpv, have to hide the dropdown, look at pictures or watch a video, and then have to reopen the dropdown. If you're like me and want to have the dropdown automatically hide, do something with a window, and then have the dropdown reappear on closing the window, you can modify your `rifle.conf` to achieve this. For example, one can use something like this for images:

`mime ^image, has sxiv, X, flag f = sxiv -a -- "$@" && hide_show termite & hide_show termite`

This will run the hide_show script once immediately when opening images with sxiv and then wait to for sxiv to be finished (quit) to run it again. However, if one also uses ranger/rifle from terminals other than the dropdown this can be a problem. To fix this, the current window can be hidden before calling sxiv and its window id saved. This is what my current command looks like:

`mime ^image, has sxiv, X, flag f = hide_show check_hide ; sxiv -a -- "$@" && hide_show check_show`

I also have this in my `rc.conf` to toggle the behaviour: `map th shell hide_show toggle_hide`

See my hide_show script for the corresponding code. It should be noted that at the time of writing, ranger's `open_all_images` setting will not work if sxiv is not at the beginning of the command. I have submitted a pull request to change this.

**_Fullscreen Toggle_**

Often dropdown terminals will provide a configurable hotkey for putting the terminal into fullscreen. Bspwm users can simply use `bspc desktop -l next` for monocle toggle or `bspc window -t fullscreen` for actual fullscreen. I have these bound in specific programs, but they can also be bound universally with sxhkd. For those not using bspwm, it may be possible to bind a key to something like `wmctrl -r :ACTIVE: -b toggle,fullscreen` instead for the same functionality. Note that, like I previously mentioned, hiding a window that is fullscreen will mess up the sizing using the xdotool method.

## Closing Notes
Don't let someone tell you to use crap like terminator over a terminal emulator you like just for native tabs and splits and don't think it's necessary to abandon your favourite lightweight terminal in favour of another just for native dropdown support!

Updated 12.16.2014 now that script uses window id
