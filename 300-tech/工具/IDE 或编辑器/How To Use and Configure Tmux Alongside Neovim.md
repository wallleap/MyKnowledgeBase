---
title: How To Use and Configure Tmux Alongside Neovim
date: 2024-09-02T02:57:21+08:00
updated: 2024-09-02T02:57:43+08:00
dg-publish: false
source: https://www.josean.com/posts/tmux-setup
---

You can find my current tmux config and the rest of my dotfiles here: [dotfiles](https://github.com/josean-dev/dev-environment-files)

## Install TMUX with Homebrew

```
brew install tmux
```

## Create ~/.tmux.conf file

```
touch ~/.tmux.conf
```

## Configure True Colors

Open ~/.tmux.conf and add the following:

```
set -g default-terminal "screen-256color"
```

## Change Default TMUX Prefix to “Ctrl-a”

Add the following to ~/.tmux.conf:

```
set -g prefix C-a
unbind C-b
bind-key C-a send-prefix
```

## Change keybinds for splitting windows

Add this to ~/.tmux.conf:

```
unbind %
bind | split-window -h

unbind '"'
bind - split-window -v
```

## Add keybind for easily refreshing tmux configuration

Add this to ~/.tmux.conf to be able to refresh tmux config with “Ctrl-a” and then “r”:

```
unbind r
bind r source-file ~/.tmux.conf
```

## Add keybinds for easily resizing tmux panes

Add these to ~/.tmux.conf:

```
bind -r j resize-pane -D 5
bind -r k resize-pane -U 5
bind -r l resize-pane -R 5
bind -r h resize-pane -L 5
```

## Add keybind for maximizing and minimizing tmux pane

Add the following to ~/.tmux.conf:

```
bind -r m resize-pane -Z
```

## Enable the mouse in tmux

Add the following to ~/.tmux.conf:

```
set -g mouse on
```

## Configure vim movements for tmux’s copy mode

Add the following to ~/.tmux.conf:

```
set-window-option -g mode-keys vi

bind-key -T copy-mode-vi 'v' send -X begin-selection # start selecting text with "v"
bind-key -T copy-mode-vi 'y' send -X copy-selection # copy text with "y"

unbind -T copy-mode-vi MouseDragEnd1Pane # don't exit copy mode after dragging with mouse
```

## Install tpm (tmux plugin manager)

Run the following:

```
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

## Add and configure tmux plugins with tpm

Add the following to ~/.tmux.conf:

```
# tpm plugin
set -g @plugin 'tmux-plugins/tpm'

# list of tmux plugins
set -g @plugin 'christoomey/vim-tmux-navigator' # for navigating panes and vim/nvim with Ctrl-hjkl
set -g @plugin 'jimeh/tmux-themepack' # to configure tmux theme
set -g @plugin 'tmux-plugins/tmux-resurrect' # persist tmux sessions after computer restart
set -g @plugin 'tmux-plugins/tmux-continuum' # automatically saves sessions for you every 15 minutes

set -g @themepack 'powerline/default/cyan' # use this theme for tmux

set -g @resurrect-capture-pane-contents 'on' # allow tmux-ressurect to capture pane contents
set -g @continuum-restore 'on' # enable tmux-continuum functionality

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
```

## You’re Done
