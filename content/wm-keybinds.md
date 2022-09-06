---
title: "Your WM keybinds aren't vim-like, are emacs-like"
date: 2022-09-06T13:56:53+01:00
draft: false
---

I've read in a lot of places people commenting that tiling windows managers offer you "vim-like" keybinds.
Those statements are somewhat inaccurate.

Vim is a modal based text editor. It relies on modes to do certain groups of actions.
Even though Emacs is also extensible and customizable, it isn't modal based.
Emacs relies on keybinds that are "chords", composed by a key with some Mnemonies and another key that act as a kind of a modifier.
[Check here the default Emacs keybinds](https://caiorss.github.io/Emacs-Elisp-Programming/Keybindings.html).

On modal based, chords aren't used. The prefixes used in Emacs keybinds are replaced by a mode and chords are replaced by noun-verb expressions.


# How would it look like

I'll give two examples to give an idea of how a modal based window manager would look like.


## Changing focused window

On most window managers, this is just done by combining the *Super* key (the default modifier used for window managing) and a direction (h, j, k, l).
On a modal based keybinds, this would be almost the same, but done in a sequence (instead of *Super* + *direction*, *Super* => *direction*).


## Moving focused

Generally, this is done by adding a modifier over the combination used to change the focused window. The most commonly used is *Shift*
(being the end result *Super* + *Shift* + *direction*).

On a modal based, we would need to create a move window mode. Let's say that the key *m* (standing to move) enters in that mode.
That would be *Super* => *m* => *direction*.
