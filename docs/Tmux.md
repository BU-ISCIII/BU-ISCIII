[tmux](https://github.com/tmux/tmux/wiki) is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen. tmux may be detached from a screen and continue running in the background, then later reattached. And even much more if you decide to play with its configuration or add some addons. For now we will stick to the vanilla flavour that you can download from their Github repository or just install it from your favourite package manager. Here is its complete [manual](http://man.openbsd.org/OpenBSD-current/man1/tmux.1).

There are many alternatives to tmux, like terminator or iTerm2, so... why should I use tmux?
* session handling: detaching from and attaching to sessions helps with context switching and remote working
* platform independence: tmux runs on every machine and OS, even without a GUI, so you can use it everywhere!
* customizable: tmux is highly configurable, so you can easily adapt it to your working style and then modify it as your skills and needs change. 

### Warning:
If you experiment some graphic issues under tmux while working with a personalised color scheme in other aplication, they may be caused by tmux not recognising your terminal emulator. Check your variable $TERM before launching tmux, and if it doesn't specify 256 colours you will have to change it by adding these lines to your .bashrc file:

`# Set 256 colors to make Tmux aware of personalised colour schemes
export TERM=${TERM}-256color
`

### Note

If you are using our personalised configuration, prefix is C-Space instead C-b.

## Basic usage

### Start tmux server
`
tmux start
`

### List all sessions
`
tmux ls
`

### Create new session named <your_session>
`
tmux new -s <your_session>
`

### Rename session
`
tmux rename-session -t <old_name> <new_name>
`

### Attach session <session_number/name>
`
tmux attach -t <session_number/name>
`

### Basic key bindings: prefix (C-b) + key
```
! Break the current pane out of the window.
" Split the current pane into two, top and bottom.
% Split the current pane into two, left and right.
$ Rename the current session.
& Kill the current window.
' Prompt for a window index to select.
, Rename the current window.
. Prompt for an index to move the current window.
0 to 9 Select windows 0 to 9.
; Move to the previously active pane.
c Create a new window.
d Detach the current client.
f Prompt to search for text in open windows.
l Move to the previously selected window.
n Change to the next window.
o Select the next pane in the current window.
p Change to the previous window.
m Mark the current pane (see select-pane -m).
M Clear the marked pane.
x Kill the current pane.
z Toggle zoom state of the current pane.
{ Swap the current pane with the previous pane.
} Swap the current pane with the next pane.
Up, Down, Left, Right Change to the pane above, below, to the left, or to the right of the current pane.
C-Up, C-Down, C-Left, C-Right Resize the current pane in steps of one cell.
Space Arrange the current window in the next preset layout.
```


## Tutorials
* https://robots.thoughtbot.com/a-tmux-crash-course
* http://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/
* https://blog.petrzemek.net/2016/02/11/my-tmux-configuration/