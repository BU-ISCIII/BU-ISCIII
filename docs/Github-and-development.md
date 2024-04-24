# Introduction

## Development

First thing first, when we start programming we need a comfortable environment. Experience makes that each one of us improves its own environment and custom configurations, which is good, but here we can share our configurations, and get some new ideas or make a good introduction to our new students. If you are already confortable with your own, feel free to use it and don't forget to share it with us

Right now, most of us use a common custom base including:

1. Select a good text editor or IDE. In my case VIM is the winner, it is fully configurable, with an incredible community with tones of plugins, and the BEST of it is that it runs just in your terminal. Working with multiple ssh connections is the easiest, all of them with the same configuration just copying one file (.zshrc) into your home.
2. ZSH configuration for a optimal use of the Linux terminal. ZSH next to OH-MY-ZSH plugin allows full terminal configuration, with many color and prompt information customization, as well as github plugins, and path autocomplete.
3. [tmux](https://github.com/tmux/tmux/wiki) is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen

## What is zsh?

Zsh is a shell designed for interactive use, although it is also a powerful scripting language. It is very similar to bash but with more capability for custom configuration. Also is used by oh-my-zsh framework with makes the use of the terminal much more friendly.

## Vim or Emacs?

Sorry, I swear I did not want to start the flame. Forget the question and just say we use Vim.

## What is a terminal multiplexer?

A terminal multiplexer is a software application that can be used to multiplex several separate pseudoterminal-based login sessions inside a single terminal display, terminal emulator window, PC/workstation system console, or remote login session, or to detach and reattach sessions from a terminal. It is useful for dealing with multiple programs from a command line interface, and for separating programs from the session of the Unix shell that started the program, particularly so a remote process continues running even when the user is disconnected.

## Version control

Version control systems are a category of software tools that help a software team manage changes to source code over time. Version control software keeps track of every modification to the code in a special kind of database. If a mistake is made, developers can turn back the clock and compare earlier versions of the code to help fix the mistake while minimizing disruption to all team members.

## Why is version control so important?

When working in a software project (or anything more sophisticated than a dirty script) development, keeping track of your work and changes is your first need. Your second one will be being able to keep different versions of it at the same time, so at least one stable version is available at all times while you experiment with new features. Finally, but not less important, when more than one person work in the same project, each of them may work in the same or different parts of the same code simultaneously, and then everything has to be merged back together without errors.

Version control systems take charge of all that hustle for us, allowing us to focuss in development and don't waste our time.

## Which version control system should I use?

We keep all our repositories in GitHub and we use git-flow as branching model, so git is the obvious way to go.

## Getting started

You can start using all this elements using the BU-ISCIII config files. Installing them is the easiest following the instructions in the repository [dotfiles](https://github.com/BU-ISCIII/dotfiles).

And of course continue reading the docs!
