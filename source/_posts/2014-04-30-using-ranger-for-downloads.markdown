---
layout: post
title: "Using Ranger as a Save Dialog"
date: 2014-04-30 23:25:27 -0400
comments: true
published: true
categories: ranger browsing firefox pentadactyl terminal tui mouseless linux downloads uploads
---

### Quick Round of Applause for Octopress
Today I found the perfect excuse to start a blog when I stumbled across [github pages](https://pages.github.com/) and [octopress](http://octopress.org/). It's something I've been wanting to do for a while, but I never found blogger or wordpress to be that attractive. Finding out that I could easily host a blog on github and write all my posts in markdown was enough to git me to do it immediately. For a first post, I thought I'd just start with presenting something basic I did recently to start using ranger for saving downloads.

## Why Ranger?
[Ranger](https://github.com/hut/ranger) is the most efficient file manager I have come across. It already integrates easily enough with most other programs. The browser is an exception and one due not to any fault of ranger. You have very little choice in general when it comes to saving files using most GUI browsers. The best the default file saver has to offer you is *`Recent Folders`*, rudimentary tab completion, and the ability to open in the last directory you saved in. While it is possible to change the file picker or file saver, none of the few options (e.g. the GTK file picker for GNOME and the QT file picker for KDE) are particularly sophisticated.

Ranger is operated entirely with the keyboard. You can press `''` to get to the last visited directory or `'<key>` to visit any directory you've bookmarked with a letter. This makes navigating even large directory structures fairly quick. When you don't have a bookmark for the exact directory you want to move a file to, you can hit `f` and then type the name of the directory. As soon as the keys you type correspond to a unique directory, that directory will automatically be opened. Alternatively, there is [fasd integration](https://github.com/hut/ranger/wiki/Commands#visit-frequently-used-directories) allowing you to jump immediately to a frequently/recently used directory. You can even [create a fzf command](https://github.com/hut/ranger/wiki/Commands#fzf-integration) to jump to a directory with a fuzzy search (e.g. using locate or find). Because of these features, using ranger can reduce a lot of scrolling and clicking to a few keypresses.

## Implementation
You can [write your own save dialog](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XUL/Tutorial/Open_and_Save_Dialogs). I don't know if it would be easy or even possible to write a save dialog wrapper for ranger. Because I often want to save without anything popping up (e.g. when saving related files that will be moved to the same directory), I just bypass the save dialog entirely and download everything automatically to a single folder. I used to just open ranger in this folder later and move things, but this led to a pileup of unorganized crap since I rarely got around to cleaning the folder.

Now I use [pentadactyl](http://5digits.org/pentadactyl/) to "replace" the default save dialog with ranger. Pentadactyl already eliminates the need for the GUI save dialog with `:w` for saving webpages (or whatever else the current url is) and `;s` for download hinting. Path and file names can be specified with tab completion as well. Other vim-oriented browsers have similar options, but this method still isn't nearly as quick as using ranger. An autocommand can be used to fake ranger integration:

    au DownloadPost * silent !/path/to/script "<file>"

With this in place, every time something is downloaded, pentadactyl will pass the full path of the downloaded file to the specified script. Making the command silent helps to ensure that pentadactyl will not steal the focus back from ranger.

Here's an example script to open ranger with the file cut:
```
	termite --geometry="600x400" -e "ranger --selectfile=""$1"" --cmd=cut" &
```

*It will look something like this (updated 2016-01-02)*:
{% img /images/ranger_file_saver.png %}

My full script can be found [here](https://github.com/noctuid/dotfiles/blob/master/scripts/bin/dl_move) and fixes a lot of problems that can occur. For example, it uses detox to clean up the filename and ignores files saved under `/tmp` (happens when using "open with"). It also supports consecutively saving files to a temporary directory and then moving them all at once later. This is useful for preventing a lot of popups when saving related files (e.g. using a count or `:tabdo`). It has an example for automatically moving certain filetypes; I do this with torrent files. Finally, it gives an example way to use emacsclient and dired instead of ranger (which can open a lot quicker in some situations).

The full script also prevents multiple popups that would normally happen due to a [pentadactyl bug](https://github.com/5digits/dactyl/issues/29). The more firefox windows opened, the more ranger instances are started. For a while, I thought this was because pentadactyl has no [autocmd clearing mechanism](https://groups.google.com/forum/#!topic/pentadactyl/rwrGG4njsq8). Sourcing an autocommands file with `au Enter` won't fix this though. This bug happens because the DownloadPost autocmd will execute for every open window. I realized this because the following would still execute from a private window when called from the autocommand but not manually:
```
command! -nargs=1 maybe-open-ranger <<EOF
if !PrivateBrowsingUtils.isWindowPrivate(window)
	silent !~/bin/ranger/ranger_browser_fm.sh <args>
endif
EOF
```

## Uploads
Pentadactyl also provides a way to avoid using a GUI for selecting a file for upload. If you can hint an upload button, pentadactyl will give you an `Upload: ` prompt and allow you to select a file with tab completion. Alternatively, it is possible to yank the file path in ranger beforehand and paste this into either the pentadactyl or GUI save dialog. As for creating a new file picker that somehow wrapped ranger, I have no clue how difficult this would be.

For some websites, I'd rather use a CLI program. For example, tumblr also allows to [post by email](https://www.tumblr.com/docs/en/posting) and even use markdown to do so. There is also [tumblr-rb](https://github.com/mwunsch/tumblr). The [imgur-cli](https://code.google.com/p/imgur-cli/) python script is useful for imgur uploads, and I've added a keybinding for it in ranger. [Imup](https://github.com/Profpatsch/imup) is pretty good for anonymous (no account login) uploads to different image hosts as well. There is also [plowshare](https://github.com/mcrapet/plowshare) for uploading to sites like mediafire.

I have not tried the following programs, but they may also be of use. For youtube, there are [youtube-upload](https://code.google.com/p/youtube-upload/) and [googlecl](https://code.google.com/p/googlecl/). Googlecl is a lot more popular and does a lot more than just youtube (e.g. picasa uploads), and youtube-upload looks like it might require you to enter your password in the command. Googlecl is also in the official arch repos. For facebook, [fbcommand](https://github.com/dtompkins/fbcmd) allows uploading photos (and multiple photos at once as an album... as does googlecl with picasa by the way).
