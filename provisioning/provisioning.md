# Provisioning

Setting up a new computer is the most painful part of my ramp-up process. It's absolutely necessary, takes too long, and effectively stands in the way of my getting started. So, I have most of my standard tooling in [the Provisioning repository][provision repo], which is private for obvious reasons.

So I'm going to talk about the various components of my standard provisioning here, so that it may be helpful to anyone who doesn't have access to my Provisioning repo.

# Aliases

I usually put all my shell aliases in a `.aliases` file so that they remain portable across shells. An alias is a nickname for a shell command. [Here's a good primer on what these are][what are shell aliases]. Here are some aliases I've found to be most helpful:

```shell
#============== Shell Customizations ==============
alias basher="vim $HOME/.zshrc"
alias shell="source $HOME/.zshrc"
alias als="vim $HOME/.aliases"
alias funcs="vim $HOME/.functions"

alias ls='ls -G -1 -p'
alias ll='ls -alF'
alias la='ls -A -p'
alias wcl='wc -l'

alias mv='mv -v'
alias cp='cp -v'
alias rm='rm -v'

alias socks='ps aux | grep'
#============= / Shell Customizations =============

#============== System Functions ==============
alias please='sudo'
alias bye='exit'
alias snore='sudo poweroff'
alias klar='clear'
alias clar='clear'
#============= / System Functions =============
```

# Functions

I usually put all my shell functions in a `.functions` file so that they remain portable across shells. Here are some functions I've found to be most helpful:

```shell
function mkcdir() {
    mkdir -p -- "$1" &&
    cd -P -- "$1"
}


function insort() {  # sort a file in-place
    sort "$1" -o "$1"
}


function headers() {  # get the headers of a CSV file and their column numbers
    if [[ $# == 1 ]]; then
        sep=','
    else
        sep=$2
    fi

    head -n1 "$1" | tr "$sep" $'\n' | cat -n
}
```

# .vimrc

I find Vim to be an awesome text editor (I'm not going to get into the flame war about emacs, etc, but I'll add the customary XKCD comic on the topic).

![](https://xkcd.com/378/)

```
set tabstop=4
set softtabstop=4
set nu
syntax on
set expandtab
inoremap <Tab> <Space><Space><Space><Space>
set softtabstop=4 expandtab
hi Comment ctermfg=3
hi String ctermfg=Green
hi Search cterm=None ctermfg=None ctermbg=red

set cursorline
hi CursorLine   cterm=NONE ctermbg=darkblue ctermfg=white guibg=darkred guifg=whit
```

This config solves the following problems with Vim that I've had over the years

1. I don't know what line number I'm on. This is a problem when I'm debugging python code and need to look at code on specific lines. Sure, I can use the `:N` syntax to go to line `N`, but that does not localize me at my current cursor. The solution is `:set number` (or `:set nu`)
2. Syntax highlighting is not turned on automatically
3. The syntax highlighting's color scheme is not ideals for me (I've reset the string, comment, and search highlight colors)
4. Especially with longer lines, it becomes difficult to tell exactlt which line my cursor is on. `set cursorline` and `hi CursorLine` solve this problem
5. Tabs vs Spaces, particularly with python code. `inoremap` helps replace tabs with spaces as I type, but also requires that I backspace four times if I need to delete the space-expanded tab. The tabstops help with this.

# .gitconfig

Just as I have shell aliases and vim configs, I'd like to have default configs and aliases for my git environment. Here's a minimally redacted version of my `~/.gitconfig` which makes my life a lot easier:

```
[user]
	name = My Name
	email = my@email.addresss
[alias]
    co = checkout
    com = checkout master
    cob = "!f() { git checkout -b $1; git push -u origin $1; }; f"
    compush = "!f() { git commit -m \"$1\"; git push ; }; f"
    pushcom = compush
    comm = commit -m
    st = status
```

A few things to note here:

1. The `"!f() {...}` syntax defines a function. Be careful though - it's a foot-gun, so tread lightly and remember "Fools rush in where angels fear to tread"
2. `cob` does `checkout -b` and pushes the (new) branch to origin
3. `compush` does `commit -m` and then pushes the commit. The only way to stop this process is to leave an empty commit file

# ~/.ssh/config

Linuxize has a really good tutorial on SSH config files [here][linuxize SSH config]. I used to keep my SSH aliases in the `.aliasses` file, but that suffers from several limitations:

1. It is an SSH alias and nothing else
2. Altering SSH parameters requires the alias to be minimal, making any parametrized interaction very tedious
3. Anything that uses SSH under the hood (eg: `scp`) will be unaware of the configurations

It is instead much easier to use `~/.ssh/config` per supported behavior, to setup SSH configurations. Here's my redacted SSH config:

```
Host my_nickname_for_the_remote_host
    User my_username_on_the_remote_host
    HostName ip_address_or_other_network_identifier
    Port 22
```

While I've only specified `User`, `HostName`, and `Port` in this config, it can also take the path to a private key file, or any other SSH config option that fits `ssh -o`.

# tmux and TPM

I used to use screen as a virtual terminal, but now I use the more powerful tmux, and it's plugin manager TPM. TPM's default config is unhelpful, so here's the config that I find to be more helpful:

```
# Start windows and panes at 1, not 0
set -g base-index 1
set -g pane-base-index 1
set -g default-terminal "screen-256color"

# Enable mouse integration (navigate through panes and windows by clicking;
# enable scrolling in a pane using the mouse scrolling)
set -g mouse

# Set vi-style keybindings
set-window-option -g mode-keys vi

# Set statusbar position to the top of the screen
# set-option -g status-position top

# Set delay between repeating or chaining tmux commands to 0
set-option -g repeat-time 0

# Removes ESC delay
set -sg escape-time 0

# List of plugins (requires TPM)
set -g @tpm_plugins '                           \
    caiogondim/maglev                           \
    tmux-plugins/tpm                            \
    tmux-plugins/tmux-sensible                  \
    tmux-plugins/tmux-resurrect                 \
    tmux-plugins/tmux-continuum                 \
    tmux-plugins/tmux-yank                      \
    tmux-plugins/tmux-pain-control              \
    tmux-plugins/tmux-copycat                   \
    tmux-plugins/tmux-open                      \
    tmux-plugins/tmux-prefix-highlight          \
'

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run -b '~/.tmux/plugins/tpm/tpm'

# remap prefix from 'C-b' to 'C-a'
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

set -g @continuum-restore 'on'
```

# Shell

I also use [zsh][zsh] with [.oh-my-zsh][omz] and the agnoster theme, which I've heavily customized.

# Putting it All Together

While all this tooling is good, it helps to automate the process of installing all of this to minimize the amount of human intervention required. So it's best to put all of this in a single shell script to be executed on the target machine. I have such a script in my [Provisioning repository][provision repo] for Ubuntu as well as Mac OS X.

[provision repo]: https://github.com/inspectorG4dget/Provisioning
[what are shell aliases]: https://shapeshed.com/unix-alias/
[linuxize SSH config]: https://linuxize.com/post/using-the-ssh-config-file/
[omz]: https://ohmyz.sh/
[zsh]: https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH