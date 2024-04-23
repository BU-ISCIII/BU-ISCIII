# Index
1. [I can't copy data from the HPC to my workstation.](#I-can't-copy-data-from-the-HPC-to-my-workstation.)
2. [Keep ssh sessions alive](#Keep-ssh-sessions-alive)
3. [Decent working environment in Windows 10 WSL](#Decent-working-environment-in-Windows-10-WSL)
4. [Copy and export only visible cells in libreoffice calc](#Copy-and-export-only-visible-cells-in-libreoffice-calc)
5. [I've installed docker and now I can't connect to my machine with ssh](#I've-installed-docker-and-now-I-can't-connect-to-my-machine-with-ssh)

## I can't copy data from the HPC to my workstation.
Take notice than ssh connection to HPC is unidirectional, I mean you can only access the HPC from your workstation, and you cannot access your workstation from the HPC. This means you have to copy date from your workstation to the HPC and **NOT** backwards.

## Keep ssh sessions alive
Asterix will kick you out every time you keep your connection idle for a set period of time. To avoid this annoying feature of the firewall, you can modify your ssh connection settings to send NULL packages every X seconds, which will keep your connection up and running. 

How you do it, depends on the tool you are using for ssh-ing. I will explain the modifications you have make in the most broadly used tools to send a keep-alive signal every 5 minutes.

### SHH

* If you have admin permissions on your machine (recommended):

Add the following lines to your "/etc/ssh/ssh_config" file

```
# Keep alive
Host *
    ServerAliveInterval 300
    ServerAliveCountMax 2
```

* If you don't have admin permissions (not tested):

Same as above, but you will have to modify your personal config file (usually "~/.ssh/config")

Now restart ssh service:
```
sudo service ssh restart
```

### PUTTY

In your session properties, go to Connection and under Sending of null packets to keep session active, set Seconds between keepalives to 300.

## Decent working environment in Windows 10 WSL
You have [here](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Decent-working-environment-in-Windows-10-WSL) a great tutorial for that.

## Copy and export only visible cells in libreoffice calc
Filtered and hidden cells are always copied with libreoffice calc. In order to copy only selected cells, you need to install an addon: [Copy only visible cells](https://extensions.libreoffice.org/extensions/copy-only-visible-cells).

When exporting only visible cells to another format (i.e. tab or csv), first copy only visible cells to a new empty sheet and then export that sheet to the new format, or all hidden cells will be exported too.

## I've installed docker and now I can't connect to my machine with ssh
Docker uses a bridge network configuration by default in order to isolate containers from the host network. Unfortunately by default uses a 172.17.0.1/16 ip range that conflicts with IPs already in use in the ISCIII network.
You need to change this default range creating a file in `/etc/docker/daemon.json` with this info (the IP range doesn't matter as long as don't conflict with any IP in the network):
```
{
 "bip": "192.168.1.5/24", 
 "fixed-cidr": "192.168.1.5/25", 
 "default-address-pools":
 	[ 
	 { "base":"192.168.2.5/24","size":28 } 
        ] 
}
```

## Boot partition is full?
Sometimes boot partition gets full because old/new kernel images are being installed on it. This artifact may happen after kernel updates or pre-installed distributions on your machine. Ideally you can add more memory to your /boot partition. Here you will see a workaround that will freed up your `/boot` partition in case you cannot resize.  

In order to safely clean `/boot` you should find the number of kernel-packages you have installed and delete all but the kernel that is currently running on your machine.   

### List installed packages that match a linux-image pattern:

```shell
# List packages installed
dpkg -l linux-image*


Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                   Version               Architecture Description
+++-======================================-=====================-============-=================================
un  linux-image                            <none>                <none>       (no description available)
rc  linux-image-5.19.0-32-generic          5.19.0-32.33~22.04.1  amd64        Signed kernel image generic
rc  linux-image-5.19.0-43-generic          5.19.0-43.44~22.04.1  amd64        Signed kernel image generic
ii  linux-image-5.19.0-45-generic          5.19.0-45.46~22.04.1  amd64        Signed kernel image generic
ii  linux-image-5.19.0-46-generic          5.19.0-46.47~22.04.1  amd64        Signed kernel image generic
```

This [web-site](https://linuxprograms.wordpress.com/2010/05/11/status-dpkg-list/) explains how to interpret the two-code letters in the first column. 

### Identify the kernel that your machine is running
Most of the time the active kernel you are using is the package with the latest versions. **DON'T REMOVE THIS PACKAGE**. You will need de active-kernel version to reboot your machine. To make sure which kernel-version your machine is running: 

```shell
uname -r

Ubuntu 5.19.0-46.47~22.04.1-generic 5.19.17
```

### Delete old kernels

```shell
# Remove old kernel packages
sudo apt-get remove linux-image-2.6.32-{32,43,45}-generic

# Remove old-dependencies
sudo apt-get autoremove
```

Check the logs, and verify that the removal was completed successfully. `dpkg -l linux-image*`


If the older kernel versions continue to appear, this means that files related to the old-kernel remains in your system. 
Then purge them:

```shell
sudo apt-get purge linux-image-5.19.0-{32,43,45}-generic
```

And list the kernel's versions again:  `dpkg -l linux-image*`

Hopefully, you will only see listed the kernel version your machine is currently running and your `/boot` partition should have freed up some disk space: `df -h`

Source: [jbgo](https://gist.github.com/jbgo)/[free-space-on-boot-disk.md](https://gist.github.com/jbgo/5016064)
