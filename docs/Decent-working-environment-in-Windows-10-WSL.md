# Windows WSL

FYI, since Windows 10 came out there is something called WSL (Windows Subsistem for Linux). This is nothing else than a minimal Ubuntu running nativelly alongside Windows 10, so you can work as you where using a Linux machine, and therefore use the same developing tools. If you do not know how to install it, check [this tutorial](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/).

But let's be clear about the main issue here: Windows terminal emulator SUCKS, and despite there are other third party software that deliver a better experience, none of them are even close to your simpliest Linux terminal emulator. If you are a putty user, you can work arround this by starting an ssh server in your ubuntu subsystem and ssh-ing into it, but... You have a full Ubuntu system in your computer, so why not try to use it at is best with only native applications?

In this guide I will explain how to install a native light weight terminal emulator and configure it to run without needing that annoying crap Windows calls terminal.

## Install X11

Your minimal Ubuntu subsystem does not include a graphic interface, so we need start by fixing it. We can install X11 in Windows 10 with [VcXsrv](https://sourceforge.net/projects/vcxsrv/):

1. Download and run the latest installer
2. Locate the VcXsrv shortcut in the Start Menu
3. Right click on it
4. Select More>Open file location
5. Create a shorcut to the VcXsrv.exe file, not the launcher, and remove the old shortcut in your desktop
6. Paste the shortcut in %appdata%\Microsoft\Windows\Start Menu\Programs\Startup
7. Edit the shotcut with these paramenters: `"C:\Program Files\VcXsrv\vcxsrv.exe" :0 -ac -terminate -lesspointer -multiwindow -clipboard -wgl`
8. Launch VcXsrv for the first time

**You may receive a prompt to allow it through your firewall. Cancel/deny this request! Otherwise, other computers on your network could access the server.**

## Configure bash to use the local X server

1. In bash run:

`echo "export DISPLAY=localhost:0.0" >> ~/.bashrc`

2. To have the configuration changes take effect, restart bash, or run:

`. ~/.bashrc`

## Install needed Ubuntu packages

`
sudo apt-get install xfce4-terminal x11-apps
`

## Create a VBS launcher for xfce4

Open a new empty file in your windows notepad and copy the following lines, then save is as "launch_terminal.vbs":

```shell
Set WshShell = CreateObject("WScript.Shell" ) 
WshShell.Run "C:\Windows\System32\Bash.exe -ic xfce4-terminal", 0 
Set WshShell = Nothing 
```

Create a shortcut in your desktop and put it a prety icon of your choice. Now you can pin it to the taskbar (chage the target of the shortcut to `wscript "C:\path\to\launch_terminal.vbs"` or you won't be able to do it), feel free to remove the shortcut, and it is done!

## Launch at home directory and fix permissions

Add these lines to your ./bashrc file:

```shell
# Start at home directory
if [ -t 1 ]; then  
  cd ~
fi 

# Fix WSL permissions                                                                                                                                               
umask 022  
```

## Better colour schemes

<https://github.com/chriskempson/base16-xfce4-terminal>

Then click on "Edit -> Preferences" and set it to your likes.

## Documentation

* <https://www.reddit.com/r/bashonubuntuonwindows/comments/64rtcl/favorite_terminal_emulator_for_wsl/>
* <https://blog.ropnop.com/configuring-a-pretty-and-usable-terminal-emulator-for-wsl/#installinganxserver>
* <https://seanthegeek.net/234/graphical-linux-applications-bash-ubuntu-windows/>
