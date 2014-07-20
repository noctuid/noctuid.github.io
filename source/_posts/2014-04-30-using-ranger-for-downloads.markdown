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

Unsurprisingly, the gui save dialog has been unflinching and apathetic in spite of all my protesting. The obvious solution is to just stop using it and have things download automatically to the `~/Downloads` folder, for example, which is what I did for some time. It's quite easy after something has downloaded to open a terminal, type a two letter alias to `cd` to that folder, `&& ranger`, and then cut and move the file. It doesn't take much time, but there are still intermediate steps that can be eliminated. Also, when I did this, I found myself never actually getting to moving the files to their appropriate locations resulting in aforementioned pile up of unorganized crap.

My solution uses [pentadactyl][4] to "replace" the default save dialog with ranger. Pentadactyl already eliminates the need for the gui save dialog with `:w` for saving webpages (or whatever else the current url is) and `;s` for download hinting. Path and file names can be specified with tab completion as well. Pentadactyl isn't the only way to do this (dwb, for example, has download hinting as well I believe), but we can do better still. Despite its faults, pentadactyl is still the most feature filled program of its kind, and I'm not familiar with any other addon or browser (vimperator excluded of course) that allows for autocommands or hooks of some sort. Because pentadactyl has autocommands, we can do something like this:

    au DownloadPost * :silent !~/bin/ranger/ranger_browser_fm.sh "<file>"

With this in place, every time something is downloaded in firefox, pentadactyl will pass the path/filename to a shell script. We can have this shell script then open ranger with that file already cut and ready to be moved. Making the command silent ensures that pentadactyl will not steal the focus back from ranger.

Here's an example shell script:
```
	no_ap=$(echo "$1" | sed "s/'//g")
	mv "$1" "$no_ap"
	escaped=$(printf '%q' "$no_ap")
	bspc rule -a termite -o floating=true center=true
	termite -e "/bin/zsh -c 'xdo resize -w +300 && xdo move -x -150 && xdo resize -h +200 && xdo move -y -100 && ranger --selectfile=""$escaped"" --cmd="cut"'" &
```

The first line just strips the file name of apostrophes which will cause problems if left in. You may have trouble with other symbols. If so, you can either make sure not to include them in the filename when saving or remove them automatically in the script. The second line just renames the saved file to the version without apostrophes. The third line takes the /Path/To/File and escapes anything that needs to be escaped (spaces\ for\ example, though spaces won't pose a problem). The fourth line is specific to my window manager (bspwm) and sets things up so that the next time the terminal emulator termite is opened, it will be floating and centered. The last line opens termite, resizes it using xdo (which is not necessary and can be removed as well), and then opens ranger, selecting and cutting the file that was just downloaded.

#### This will look something like this (I went up to show that the file was successfully cut):
{% img /images/ranger_file_saver.png %}

Here's where we run into a problem. Like a few features of pentadactyl (it would seem), implementation of autocommands might be seen as half baked. Unlike in vim, there is no such thing as `augroup`, and there is no way to clear autocommands. If you put this autocommand in your `~/.pentadactylrc`, it will be executed every time you open a new window, meaning if you have three windows open and save a file, you'll get three ranger popups. I originally tried solving this by having the script only execute if the current window did not have the class Termite, but because of the timing, this doesn't work well. The best way I've found to deal with autocommands because of this problem is, well.. with another autocommand (in the ~/.pentadactylrc):

	au Enter * :source ~/.pentadactyl/autocommands

This will source a file with your autocommands only on firefox startup, ensuring that your autocommands are executed only once without having to manually add them every time. For more information and my actual config files, see my [dotfiles][5] repo.

## Update (5.30.14) 
I thought that it also might be useful for this as well as for any other scripts executed with autocommands to differentiate between different downloads. For example, it's easy enough to distinguish between different filetypes by examining what comes after the final period in the filename. This allows for the possibility of doing things like moving certain filetypes from certain websites to specific folders before or without ever opening ranger. You may not _always_ want to use ranger after saving the files. For example, you may not want to use ;s and ranger if you're saving from a private window. In this case, it's just a matter of differentiating between a private and normal browsing session:

	window_title=$(xtitle)
	browser_type=$(echo "${window_title##*-}")
	if [ "$browser_type" == " Pentadactyl" ]; then
		previous code
	fi

Because browser_type will be " Pentadactyl (Private Browsing)" if it is a private window (I have my titlestring set to "Pentadactyl"; I believe the default is "Firefox"), ranger won't be opened after the download. [Xtitle][6] is in the aur or can be easily installed without using the aur. I just used it because it was easy and I have it installed for bar-aint-recursive already; alternatively, xprop would work fine as well. Just grep for WM_NAME(STRING), for example, and take it from there.

There may also be other specific situations in which it would be desirable to temporarily stop a ranger window from appearing. For example, when batch downloading files that will all end up in the same folder (i.e. wallpaper, related , etc.), it's inneficient to take the time to move each file individually. I've just done this by creating a file when I want to turn it off:

	mkdir -p ~/bin/ranger
	# toggle script
	if [ "$1" == "toggle" ]
	then
		if [ -f ~/bin/ranger/off.txt ]
		then
			rm ~/bin/ranger/off.txt
		else
			touch ~/bin/ranger/off.txt
		fi
	else
		if [ ! -f ~/bin/ranger/off.txt ]
		then
			previous code 
		fi
	fi

Now ranger won't execute when toggled off. I use this mostly when downloading related images after a google image search or when downloading comics/galleries using dta and the webcomic reader userscript. You can bind this in pentadactyl like so (replace with location of the script):

	nmap -ex <leader>r silent !~/bin/ranger/ranger_browser_fm.sh toggle

Just toggle it back on when you want to move all the files at once. If any of that is confusing, just see the full actual script in my dotfiles repo (bin/ranger/ranger_browser_fm.sh).

## Uploads
Pentadactyl also provides a way to avoid using a gui for selecting a file for upload. If you can hint an upload button, pentadactyl will give you an `Upload: ` prompt and allow you to select a file with tab completion. Alternatively, it is possible to yank the file path in ranger beforehand and paste this into either the pentadactyl or gui save dialog.

For some websites, I'd rather use a cli program. I've used [tumblr-rb][7] for tumblr uploads. Tumblr also allows you to [post by email][8] and even use markdown to do so. The [imgur-cli][9] python script is useful for imgur uploads. [Imup][10] is pretty good for anon uploads to different image hosts as well. I'm using mutt for email, so attachments are easy.

I have not tried the following yet, but they may also be of use. For youtube, there are [youtube-upload][11] and [googlecl][12]. Googlecl is a lot more popular and does a lot more than just youtube (i.e. picasa uploads), and youtube-upload looks like it might require you to enter your password in the command. Googlecl is also in the official arch repos, so I'd recommend it over youtube-upload. For facebook, [fbcommand][13] allows uploading photos (and multiple photos at once as an album... as does googlecl with picasa by the way). If you actually do direct uploads to twitter, I'm not sure if there is any existing program for this. There is [t][14], but I don't think it allows for uploads. Another program that would be nice for me would be one for deviantart because uploading to deviantart is a pain in the ass and could be so much faster with categories and such scripted (though the complexity is probably why there isn't one.. at least as far as I know).

[1]: https://pages.github.com/
[2]: http://octopress.org/
[3]: https://github.com/hut/ranger
[4]: http://5digits.org/pentadactyl/
[5]: https://github.com/angelic-sedition/dotfiles
[6]: https://github.com/baskerville/xtitle
[7]: https://github.com/mwunsch/tumblr
[8]: https://www.tumblr.com/docs/en/posting
[9]: https://code.google.com/p/imgur-cli/
[10]: https://github.com/Profpatsch/imup
[11]: https://code.google.com/p/youtube-upload/
[12]: https://code.google.com/p/googlecl/
[13]: https://github.com/dtompkins/fbcmd
[14]: https://github.com/sferik/t
