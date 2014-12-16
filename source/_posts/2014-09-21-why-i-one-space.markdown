---
layout: post
title: "Why I One-Space"
date: 2014-09-21 23:46:12 -0400
comments: true
published: true
categories: writing
---

[Steve Losh's article](http://stevelosh.com/blog/2012/10/why-i-two-space/) on using two spaces after a sentence continues to popup, going mostly unchallenged it would seem. I personally fail to see any reason to use two spaces that isn't completely subjective and find his arguments otherwise to be self-defeating.

## His Arguments for Two Spaces
1. It looks better when using a monospaced font.
2. It gives the user more "power"

## Two-Spacing is Pretty
This is entirely subjective. I use monospaced fonts all the time and still think two spaces in between a sentence looks hideous.

## Two-Spacing Gives More Power?
Okay so what about the non-trivial argument? Well, first of all, it seems to be the same as the argument he says is not convincing (less effort). His example for "power" only applies to vim users and only to some of them.

This is what happens when I type `das` in vim using his example.

Before with cursor above `t`:

`Bob started speaking. Hello, Mr. Smi[t]h! How are you today?`

After `das`:

`Bob started speaking. How are you today?`

It ends up looking exactly how I want. This is because I'm using reedes' excellent [textobj-sentence](https://github.com/reedes/vim-textobj-sentence) plugin that makes vim's `as` and `is` smarter. It even allows for adding custom abbreviations to ignore. As Losh says, we can have our cake and eat it too!

Granted, this plugin didn't exist at the time of the article's writing (though the idea isn't exactly hard to come up with). Even if this plugin did not currently exist, his argument is bunk. What is the benefit of being able to do things like `das`? If what is meant by power is not efficiency and saving keystrokes, effort, and time, then I've no clue what the point is. Count how many times I've used abbreviations in this post. You don't even have to count the number sentences to realize the number of extra keystrokes I would have had to type to put two spaces in between every sentence is exponentially greater than the single extra keystroke it would have taken me to deal with that abbreviation if I had needed to (the dot operator). So even if space is only half a percent of keystrokes typed, the number of keystrokes needed to deal with abbreviations is even more insignificant. So much for power.

As for parsing, this is merely a hypothetical. Has he ever needed to do that? I actually often use a script for counting sentences, and it's really not that difficult to deal with single spaced sentences like textobj-sentences does. A script that chose not to deal with it would suffer from the problem of not working with anything written by people who don't two-space.

The only thing two-spacing seems be to me is a bad habit.
