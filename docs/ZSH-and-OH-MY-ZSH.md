# ZSH terminal
Zsh is a shell designed for interactive use, although it is also a powerful scripting language. It is very similar to bash but with more capability for custom configuration. Also is used by oh-my-zsh framework with makes the use of the terminal much more friendly.
For some comparison with bash you can read [this article](http://stackabuse.com/zsh-vs-bash/)

# oh-my-zsh
[Oh-my-zsh](http://ohmyz.sh/) is a zsh framework with hundreds of custom themes and plugins. You can check all the plugins and themes available [here](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins-Overview) and use whatever you want.

Currently I use git, git-flow and django plugins, which extends command autocomplete and adds git events decoration to the prompt which is very handy. Also I customize a theme called cleanCustom-theme.

Some other features are:
* Path completion is case insensitive.
* Path completion is eased with tab key.
* Prompt information improvement.
* Useful aliases pre-configured. For a full list of active aliases, run `alias`.

## Oh-my-zsh customization
Configuration is set in .zshrc. There you can change the theme and add all the plugins you desire.
```Bash
# Set name of the theme to load.
# Look in ~/.oh-my-zsh/themes/
# Optionally, if you set this to "random", it'll load a random theme each
# time that oh-my-zsh is loaded.
ZSH_THEME="cleanCustom"
# Which plugins would you like to load? (plugins can be found in ~/.oh-my-zsh/plugins/*)
# Custom plugins may be added to ~/.oh-my-zsh/custom/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git)
```
Also aliases can be defined. HPC connection aliases are set by default. More aliases can be configured creating a file named .aliases in your home folder.
```Bash
# Source file with software paths
if [ -f /etc/profile.d/extrapath.sh ];then
	source /etc/profile.d/extrapath.sh
fi
```