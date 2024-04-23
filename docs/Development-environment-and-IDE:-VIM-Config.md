# Why VIM?
Vim, short for Vi IMproved, is a configurable text editor often used as a Python development environment. Vim proponents commonly cite the numerous plugins, Vimscript and logical command language as major Vim strengths.

![](https://github.com/BU-ISCIII/coding_documentation/blob/master/images/vim-the-editor.jpg)

Vim's philosophy is that developers are more productive when they avoid taking their hands off the keyboard. Code should flow naturally from the developer's thoughts through the keyboard and onto the screen. Using a mouse or other peripheral is a detriment to the rate at which a developer's thoughts become code.

Vim has a logical, structured command language. When a beginner is learning the editor she may feel like it is impossible to understand all the key commands. However, the commands stack together in a logical way so that over time the editor becomes predictable.

# VIM/VIMX install
VIM is pre-installed in the major Linux distributions, BUT for some useful features as copying into vim from clipboard(LINK) Vim requires the +clipboard feature flag. Most Linux distributions ship with a "minimal" Vim build by default, which doesn't have +clipboard, so we will need to install it
* Debian & Ubuntu: Install vim-gtk or vim-gnome.
* Fedora & CentOS: install vim-X11, and run vimx instead of vim (more info).
* Arch Linux: install gvim (this will enable +clipboard for normal vim as well).


For now on we will need to use vimx command instead of vim. One handy option is create an alias in our .bashrc or .zshrc file:

```Bash
alias v="vimx"
```

# VIM configuration
## VIM config file
The central place for editing your vim configuration is your **.vimrc** file, located in your home directory or folder. This serves as the primary file for changes to your vim configuration.
A quick search in google gives us tones of .vimrc configuration mastered by vim geeks around the world. 
In this repository we have adapted [one](https://github.com/s3rvac/dotfiles/tree/master/vim) created by Petr Zemek that is great commented.

TIP: vim is a very powerful tool that can be overwhelming at first impression. My humble advice, take it slowly, just look for some 4-5 useful shortcuts and start using it.

## VIM plugins
Also in your home folder is a folder called **.vim**. In it, there are several sub-directories which hold plugins for Vim. The basic directory structure is similar to that shown below.
```
.vim
  |____after    # for overrides to system-level vim
  |____autoload # a directory for some plugins
  |____colors   # custom colorthemes
  |____doc      # documentation
  |____ftplugin # custom filetype plugins
  |____indent   # custom indentation overrides
  |____plugin   # plugin installation directory
  |____syntax   # custom syntax coloring files
```

There are a bunch of plugins for vim, and we will be extended them, but for know in our configuration we find some custom configuration:
* **Pathogen**: Manage your 'runtimepath' with ease. In practical terms, pathogen.vim makes it super easy to install plugins and runtime files in their own private directories.
* **Grubvox**: plugin that adds a fancy color scheme used in our configuration.
* **nerdtree**: a treexplorer file for vim
* **Pydiction** and **python-syntax**: plugins for tab-complete your python code.

# Useful shortcuts
You can get crazy reading vim tutorials, so I recommend here two of them that can be useful for newbies. Read them and get the basics (minimum vim modes, movements, copy/paste):

* [Easy tutorial with the very basics](https://www.howtoforge.com/vim-basics)
* [More complete tutorial](https://www.howtoforge.com/vim-basics)
* ["Things about vim I wish I knew earlier" from out new best friend Petr Zemek](https://blog.petrzemek.net/2016/04/06/things-about-vim-i-wish-i-knew-earlier/)
  
## Highlights custom configuration
In our vim configuration we have mapped our **leader key** with ",", this means that most custom shortcuts begin with ,command. For example:

* Comment the line of text where is the cursor: ,c

Other cool stuff configured just for you to use are:
* Path autocomplete (Prioritization and filtering according to file extensions)
* Color and highlight syntax.
* Trailing spaces removed when saving.
* Tabs expanded as spaces in python.
* Highlight for error indenting and spacing.

## Nerdtree

* ,n: Start nerdtree
* ,i: Open in horizontal split (with cursor on file you want to open)
* ,s: Open in vertical split (with cursor on file you want to open)
* ,t: Open in new tab
* ,m: Open menu
* ,r: Refresh tree

## Custom features
Best way to know all custom configuration is reading the .vimrc file. I know this can be overwhelming at first so I show here some useful shortcuts I use a lot:

### Function keys
* F1: Toggle spell checker.
* F2: Toggle the display of unprintable characters (expanded tabs: >-, trailing whitespace: #, ends of lines: $, and non-breakable space: ~)
* F3: Toggle line wrapping.
* ,F3: Toggle relative/absolute numbers.

### Other mappings and abbreviations
* Ctrl-e: jump to the end of the line in the insert mode.
* space: Hitting space in normal/visual mode will make the current search disappear
* ,rc: Replaces the current word (and all occurrences)
* ,cc: Changes the current word (and all occurrences).
* ,rts: Replace tabs with spaces.
* ,c: comment line
* ,u: uncomment line

# Known issues and fixes
* Change cursor shape depending on editing mode: http://vim.wikia.com/wiki/Change_cursor_shape_in_different_modes
* Binary characters appear when using gnome-terminal: https://askubuntu.com/questions/855080/gnome-terminal-vim-strange-characters
