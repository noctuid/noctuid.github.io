---
layout: post
title: "Using Ranger as a Save Dialog"
date: 2014-04-30 23:25:27 -0400
comments: true
published: false
categories: ranger firefox pentadactyl terminal tui mouseless linux
---

### Quick Round of Applause for Octopress
Today I found the perfect excuse to start a blog when I stumbled across [github pages][1] and [octopress][2]. It's something I've been wanting to do for a while, but I never found blogger or wordpress to be that attractive. Finding out that I could easily host a blog on github and write all my posts in markdown was enough to git me to do it immediately. For a first post, I thought I'd just start with presenting something basic I did recently to start using ranger for saving downloads.

## A More Universal Ranger
[Ranger][3] is most efficient file manager I have come across, and I use it for roughly 99% of file management. Ranger already integrates nicely with most other programs. The exception is the browser. As far as I'm aware, you have pretty much no choice in general when it comes to saving files. When saving normally in most browsers, you are forced to deal with the dreaded gui save dialog. The best it has to offer you is *`Recent Folders`* and the ability to open in the last directory you saved in. Forget support for bookmarks or decent keybindings; it's essentially the shittiest gui "file manager" in existence, and you don't even have the option to replace it as far as I know. I'm not sure exactly what program is responsible for this save popup on GNU/Linux, but it seems to be independent of the browser itself as it is the same for different browsers. I find disturbing the lack of information or interest I could find in this clear conspiracy. Using the default popup encourages either a pile up of your random unorganized shit in a single folder or induces insanity due to the frustration involved in getting to the right directory. I've wasted too much time in the past clicking through ten+ folders just to save one file.

Unsurprisingly, the gui save dialog has been unflinching and apathetic in spite of all my bitching. The obvious solution is to just stop using the abomination and have things download automatically to the `~/Downloads` folder, for example, which is what I did for some time. It's quite easy after something has downloaded to open a terminal, type a two letter alias to `cd` to that folder `&& ranger`, and then cut and move the file. It doesn't take much time, but there are still intermediate steps that can be eliminated. Also, when I did this, I found myself never actually getting to moving the files to their appropriate location resulting in that pile up of unorganized crap.

My solution uses [pentadactyl][4] to "replace" the default save dialog with ranger. Pentadactyl already eliminates the need for the gui save dialog with `:w` for saving webpages and `;s` for download hinting. Path and file names can be specified and with tab completion as well. Pentadactyl isn't the only way to do this (dwb, for example, has download hinting as well I believe), but we can do better still. Despite its faults, pentadactyl is still the most feature filled program of its kind, and I'm not familiar with any other addon or browser (vimperator excluded of course) that allows for autocommands. Because pentadactyl has autocommands, we can do something like this:


    au DownloadPost * :silent !~/bin/ranger/ranger_browser_fm.sh "<file>"

With this in place, every time something is downloaded in firefox, pentadactyl will pass the path/filename to a shell script. We can have this shell script then open ranger with that file already cut and ready to be moved. Making the command silent ensures that pentadactyl will not steal the focus back from ranger.

Here's an example script:
```
	no_ap=$(echo "$1" | sed "s/'//g")
	mv "$1" "$no_ap"
	escaped=$(printf '%q' "$no_ap")
	bspc rule -a termite -o floating=true center=true
	termite -e "/bin/zsh -c 'xdo resize -w +300 && xdo move -x -150 && xdo resize -h +200 && xdo move -y -100 && ranger --selectfile=""$escaped"" --cmd="cut"'" &
```

The first line just strips the file name of apostrophes which will cause problems if left in. You may have trouble with other symbols. If so, you can either make sure not to include them in the filename when saving or remove them in the script. The second line just renames the saved file to the version without apostrophes. The third line takes the /Path/To/File and escapes anything that needs to be escaped (spaces\ for\ example, though spaces won't pose a problem). The fourth line is specific to my window manager (bspwm) and sets things up so that the next time the terminal emulator termite is opened, it will be floating and centered. The last line opens termite, resizes it using xdo (which is not necessary and can be removed as well), and then opens ranger, selecting and cutting the file that was just downloaded.


#### This will look something like this (I went up to show that the file was successfully cut):
{% img /images/ranger_file_saver.png %}

Here's where we run into a problem. Like many features of pentadactyl (it would seem), implementation of autocommands seems half baked. Unlike in vim, there is no such thing as `augroup`, and there is no way to clear autocommands. If you put this autocommand in your `~/.pentadactylrc`, it will be executed every time you open a new window, meaning if you have three windows open and you save a file, you'll get three ranger popups. I originally tried solving this by having the script only execute if the current window did not have the class Termite, but because of the timing, this doesn't work well. The best way I've found to deal with autocommands because of this problem is well.. with another autocommand (in the ~/.pentadactylrc):

	au Enter * :source ~/.pentadactyl/autocommands


This will source a file with your autocommands only on firefox startup, ensuring that your autocommands are executed only once without having to manually added them every time. For more information and my actual config files, see my [dotfiles][5] repo.

[1]: https://pages.github.com/
[2]: http://octopress.org/
[3]: https://github.com/hut/ranger
[4]: http://5digits.org/pentadactyl/
[5]: https://github.com/angelic-sedition/dotfiles
