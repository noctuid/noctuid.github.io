---
layout: post
title: "Using Ranger as a Save Dialog"
date: 2014-04-30 23:25:27 -0400
comments: true
published: true
categories: ranger browsing firefox pentadactyl terminal tui mouseless linux downloads uploads
---

### Quick Round of Applause for Octopress
Today I found the perfect excuse to start a blog when I stumbled across [github pages][1] and [octopress][2]. It's something I've been wanting to do for a while, but I never found blogger or wordpress to be that attractive. Finding out that I could easily host a blog on github and write all my posts in markdown was enough to git me to do it immediately. For a first post, I thought I'd just start with presenting something basic I did recently to start using ranger for saving downloads.

## A More Universal Ranger
[Ranger][3] is the most efficient file manager I have come across. It already integrates easily enough most other programs. The browser is an exception and one due not to any fault of ranger. As far as I'm aware, you have pretty much no choice in general when it comes to saving files using most gui browsers. When saving normally in most browsers, you are forced to deal with the dreaded gui save dialog. The best it has to offer you is *`Recent Folders`*, rudimentary tab completion, and the ability to open in the last directory you saved in. Forget support for bookmarks or decent keybindings; it's essentially the shittiest gui "file manager" in existence, and you don't even have the option to replace it as far as I know. I'm not sure exactly what program is responsible for this save popup on GNU/Linux, but it seems to be independent of the browser itself as it is the same for different browsers. I find disturbing the lack of information or interest I could find in this clear conspiracy. Using the default popup can encourage either a pile up of your random, unorganized shit in a single folder or induce insanity due to the frustration involved in getting to the right directory. I've wasted too much time in the past clicking through ten+ folders just to save one file.

Unsurprisingly, the gui save dialog has been unflinching and apathetic in spite of all my protesting. The obvious solution is to just stop using it and have things download automatically to the `~/Downloads` folder, for example, which is what I did for some time. It's quite easy after something has downloaded to open a terminal, type a two letter alias to `ranger ~/Downloads` and then cut and move the file. It doesn't take much time, but there are still intermediate steps that can be eliminated. Also, when I did this, I found myself never actually getting to moving the files to their appropriate locations resulting in the aforementioned pile up of unorganized crap.

My solution uses [pentadactyl][4] to "replace" the default save dialog with ranger. Pentadactyl already eliminates the need for the gui save dialog with `:w` for saving webpages (or whatever else the current url is) and `;s` for download hinting. Path and file names can be specified with tab completion as well. Pentadactyl isn't the only way to do this (dwb, for example, has download hinting as well I believe), but we can do better still. Because pentadactyl has autocommands, we can do something like this:

    au DownloadPost * silent !~/bin/ranger/ranger_browser_fm.sh "<file>"

With this in place, every time something is downloaded in firefox, pentadactyl will pass the path/filename to the specified shell script. We can have this shell script then open ranger with that file already cut and ready to be moved. Making the command silent helps ensure that pentadactyl will not steal the focus back from ranger.

Here's an example shell script:
```
	no_ap=$(echo "$1" | sed "s/'//g")
	mv "$1" "$no_ap"
	escaped=$(printf '%q' "$no_ap")
	bspc rule -a termite -o floating=true center=true
	termite -e "/bin/bash -c 'xdo resize -w +300 && xdo move -x -150 && xdo resize -h +200 && \
		xdo move -y -100 && ranger --selectfile=""$escaped"" --cmd="cut"'" &
```

The first part just strips the file name of apostrophes which will cause problems if left in. You may have trouble with other symbols. If so, you can either make sure not to include them in the filename when saving or remove them automatically in the script. The fourth line is specific to my window manager (bspwm) and sets things up so that the next time the terminal emulator termite is opened, it will be floating and centered. The last line opens termite, resizes it using xdo (which is not necessary and can be removed as well), and then opens ranger, selecting and cutting the file that was just downloaded.

*This will look something like this (I went up to show that the file was successfully cut)*:
{% img /images/ranger_file_saver.png %}

There is another problem with this. The more firefox windows you open, the more ranger instances will be started. For a while, I thought this was because pentadactyl has no [autocmd clearing mechanism](https://groups.google.com/forum/#!topic/pentadactyl/rwrGG4njsq8). Sourcing an autocommands file with `au Enter` won't fix this though. It happens because the DownloadPost autocmd will execute for every open window. I realized this because the following would still execute from a private window when called from the autocommand but not manually:
```
command! -nargs=1 maybe-open-ranger <<EOF
if !PrivateBrowsingUtils.isWindowPrivate(window)
	silent !~/bin/ranger/ranger_browser_fm.sh <args>
endif
EOF
```

See the [pentadactyl issue](https://github.com/5digits/dactyl/issues/29) for a potential workaround. Also see my [dotfiles][5] for the full script (under `scripts/bin/ranger`) with a simpler workaround. It has extra features like adding an option to open downloads in dired (which can actually be faster than opening a new ranger session with the emacs daemon), ignoring files saved to `/tmp`, and allowing the ranger-opening behaviour to be toggled on and off. I'll also probably add functionality to sort files or take different actions based on the filetype (to ignore torrent files for example).

## Uploads
Pentadactyl also provides a way to avoid using a gui for selecting a file for upload. If you can hint an upload button, pentadactyl will give you an `Upload: ` prompt and allow you to select a file with tab completion. Alternatively, it is possible to yank the file path in ranger beforehand and paste this into either the pentadactyl or gui save dialog.

For some websites, I'd rather use a cli program. I've used [tumblr-rb][7] for tumblr uploads. Tumblr also allows you to [post by email][8] and even use markdown to do so. The [imgur-cli][9] python script is useful for imgur uploads, and I've added a keybinding for it in ranger. [Imup][10] is pretty good for anon uploads to different image hosts as well. There is also [plowshare](https://github.com/mcrapet/plowshare) for uploading to sites like mediafire.

I have not tried the following yet, but they may also be of use. For youtube, there are [youtube-upload][11] and [googlecl][12]. Googlecl is a lot more popular and does a lot more than just youtube (i.e. picasa uploads), and youtube-upload looks like it might require you to enter your password in the command. Googlecl is also in the official arch repos, so I'd recommend it over youtube-upload. For facebook, [fbcommand][13] allows uploading photos (and multiple photos at once as an album... as does googlecl with picasa by the way).

[1]: https://pages.github.com/
[2]: http://octopress.org/
[3]: https://github.com/hut/ranger
[4]: http://5digits.org/pentadactyl/
[5]: https://github.com/noctuid/dotfiles
[7]: https://github.com/mwunsch/tumblr
[8]: https://www.tumblr.com/docs/en/posting
[9]: https://code.google.com/p/imgur-cli/
[10]: https://github.com/Profpatsch/imup
[11]: https://code.google.com/p/youtube-upload/
[12]: https://code.google.com/p/googlecl/
[13]: https://github.com/dtompkins/fbcmd
[14]: https://github.com/sferik/t
