---
layout: post
title: "A More Evil Helm"
date: 2015-02-03 12:44:40 -0500
comments: true
published: true
categories: emacs helm unite hydra
---

## Modes, Submodes, States, and Now Hydras
The [hydra](https://github.com/abo-abo/hydra) plugin for emacs basically provides an extremely convenient way to create custom, persistent states where single keys will have a different effect than they normally would. The point is the same as that behind modal editing. If you're going to be performing multiple related actions in a sequence, it is more efficient to enter a state where you can execute those actions using only single keys.

I use the word "state" because the word "mode" has a very different different meaning in emacs (kind of like "yank" does). The equivalent of a vim mode in emacs is basically an evil state (using the [evil plugin](https://gitorious.org/evil/pages/Home)). The equivalent of a [vim-submode](http://www.vim.org/scripts/script.php?script_id=2467) would be a hydra. There have already been evil plugins such as [evil-lisp-state](https://github.com/syl20bnr/evil-lisp-state) and code in spacemacs that create mini-states for more specific tasks, but hydra makes the creation of these much simpler.

A commonly given example of when one might use a hydra is when repeating an action over and over, such as scrolling or zooming. However, it's generally simpler to just use the dot operator for repeating single actions. A better one would be the [example](http://oremacs.com/2015/02/03/one-hydra-two-hydra/) given by abo-abo (hydra's author) and bcarell for switching between splits.

Hydra has other convenience features, such as allowing for help text for each key or "head" in the hydra to be printed in the echo area. It does everything one would initially hope for and allows for global hydras as well as hydras in specific major modes. It even makes a "color" distinction where a head with the color blue will exit the state while those that are red (default) will not.

## A Hydra to Make Helm More Like Unite
[Helm](https://github.com/emacs-helm/helm) is pretty much the emacs equivalent of [unite](https://github.com/Shougo/unite.vim) except even more integrated. One thing I sometimes miss when using Helm is the ability to switch from `insert` to `normal` to do things like mark or move between candidates. I orginally created substates using the same method `evil-lisp-state` uses. It works, but it's ugly, long (>60 lines), and you have to define a new evil state each time. With hydra there's no evil involved, and it's as simple as this:

```
(defhydra helm-like-unite ()
  "vim movement"
  ("?" helm-help "help")
  ("<escape>" keyboard-escape-quit "exit")
  ("<SPC>" helm-toggle-visible-mark "mark")
  ("a" helm-toggle-all-marks "(un)mark all")
  ;; not sure if there's a better way to do this
  ("/" (lambda ()
          (interactive)
          (execute-kbd-macro [?\C-s]))
       "search")
  ("v" helm-execute-persistent-action)
  ("g" helm-beginning-of-buffer "top")
  ("G" helm-end-of-buffer "bottom")
  ("j" helm-next-line "down")
  ("k" helm-previous-line "up")
  ("i" nil "cancel"))
```

Then bind it to `escape`:

```
(define-key helm-map (kbd "<escape>") 'helm-like-unite/body)
```

Now you can enter the `helm-like-unite` hydra with `escape` in helm and then use `j` and `k` to navigate up and down and `space` to mark candidates. Hydra even makes numbers work as digit arguments, so you can do `9k` as you would in vim. You can use `escape` to quit helm or `i` to return to normal functionality. Keys not mapped in a hydra will exit the hydra, so `return` and `tab` will have their normal behaviour. `?` will bring up helm's help. `/` will do whatever `C-s` would do for those selected buffers (e.g. `helm-buffers-run-multi-occur` or `helm-ff-run-grep`). `v` will describe the function, or preview the buffer/file, or go to a line (in helm occur), etc. It's easy to add more actions, for example binding `p` to `helm-copy-to-buffer` or `d` and `u` to `helm-next-page` and `helm-previous-page`.

A downside could be that it now takes two escapes to get out of helm. This will probably appeal mostly only to evil users. If you don't move up and down and mark specific candidates a lot in helm, this probably isn't worth it. I still think it's a cool example of hydra's simplicity.
