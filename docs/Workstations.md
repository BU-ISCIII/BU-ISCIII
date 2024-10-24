# Workstations

BU-ISCIII workstations are part of our desktop infrastructure, besides several screens, internal HDDs and external HDDs.

An inventory with IPs, HOSTNAMES and item ids can be found in our shared resource in this [drive url](https://docs.google.com/spreadsheets/d/16Sm_l-GkpxMROQwA8NibD5tnGNc26KjOvZgzujYGdk0/edit?usp=drive_link).

## Software install

- Common software from the OS repositories can be installed using the package manager.

- Software or software versions not found in the repositories and inteded to be used by all users has to be compiled in `/opt/`, inside a folder named as the software and version. You can use any way you like to add the software to the path.

- Other software needed for testing, developing or one-time use can be installed in the local home folder of the user using [micromamba](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html).

## Mounting remote drives

See [[Data Structure]].

## Create boot start disk

Download the latest LTS version from [ubuntu.com](https://ubuntu.com/download/desktop).

In your Linux server open the "startup disk Creation" tool to create a bootable USB disk.

## Clean installation

1. Boot the server using the bootable disk.

When server is planned for computation propose, the installation settings are different to prevent for one side that critical partitions could full by user data and for other hand to install software packets for this propose.

During software installation, before running the installation GUI:

- Open gparted in your WS
    1. Create swap primary partition at the end of the disk (8000 MB). Select filesystem as ext4 and partition name and label as _swap_.
    2. Create primary partition destined to the EFI (2000 MB). Select FAT32 as filesystem, and label it _efi_.
    3. Create pvolume with the rest of disk space. Select as filesystem lvm2 pv, and as partition name and label pvolume.(Remaining space)
    4. Create vgroup and lvolumes with the command line as follows, where <vgroup> is "vg_<your_pc_name>", and <your_pc_name> is "pc" followed by the numeric code on the isciii's label of the physical machine:

```Bash
pvdisplay
vgcreate <vgroup> /dev/sda3
lvcreate -L 20G -n lv_root <vgroup>
lvcreate -L [whatever space you want to assign to your home]T -n lv_home <vgroup> # Left some space free so you can reassign this space in whatever partition you need in the future.
lvcreate -L 30G -n lv_opt <vgroup>
lvcreate -L 20G -n lv_tmp <vgroup>
lvcreate -L 20G -n lv_var <vgroup>
lvcreate -L 2G -n lv_var_log <vgroup>
lvcreate -L 20G -n lv_root_all <vgroup>
```

Then, in the installation GUI select the following options:

- Which applications would you like to start with?
  - **Minimal installation**
- Other options
  - Click on the Download updates while installing
- Click Install third-party software for graphics and Wi-Fi hardware
- Installation type menu.
  - Where would you like to install ?
    - Click on **Manual**
    - Install your favourite Linux distribution on top of those partitions (SSD if exist), assigning the following mounting points:

```Bash
efi         -> efi      (FAT32)
lv_root_all -> /        (ext4)
lv_root     -> /root    (ext4)
lv_home     -> /home    (ext4)
lv_opt      -> /opt     (ext4)
lv_tmp      -> /tmp     (ext4)
lv_var      -> /var     (ext4)
lv_var_log  -> /var/log (ext4)
swap        -> swap     (linux_swap)
```

- During software installation, select the following options:

1. _Welcome_: Select `English` in the left panel and `install Ubuntu`
2. _Keyboard layout_: `Spanish` > `Spanish`
3. _Updates and other software_: Select:
    - `Normal installation`
    - `Install third-party software`...
4. _Installation Type_: Select:
    - `Erase Ubuntu and reinstall`
    - `Install Now`
    - `Continue` in the new window
5. _Where are you_: `Madrid`
6. Who are you?:
    - Your name: `bioinfoadm`
    - Your computer's name: "PC`<inventoryNumber>`"
    - Username same as your name
    - Password: **Ask to us**
    - Require my password to log in
    - Continue

## Steps after OS installation

### Users and groups

1. Create the following groups replacing the gid values for the ones defined in axterix:

```Bash
sudo addgroup bioinfo
sudo groupmod -g <gid_bioinfo> bioinfo
sudo addgroup sshaccess
sudo groupmod -g <gid_sshaccess> sshaccess
sudo addgroup bioinfoaccess
sudo groupmod -g <gid_bioinfoaccess> bioinfoaccess
```

2. Add bioinfoadm user to the groups

```Bash
sudo usermod -a -G bioinfoaccess bioinfoadm
sudo usermod -a -G sshaccess bioinfoadm
sudo usermod -a -G bioinfo bioinfoadm
```

3. Create your user `<user>` and add the following groups

3.1 If your user name is already defined in asterix use the same uid and gid as in asterix

```Bash
sudo adduser <user> -u <uid>
sudo usermod -a -G bioinfoaccess,sshaccess,bioinfo,sudo <user>
sudo groupmod -g <gid> <user>
```

3.2 If you do not have user account in asterix yet, do the following:

```Bash
sudo adduser <user>
sudo usermod -a -G bioinfoaccess,sshaccess,bioinfo,sudo <user>
```

3.2.1 If you get your user account in asterix after some time using your server, perform the following:
Use `id <user>` logged in asterix to know your assigned uid and gid, and change them in your local machine:

```Bash
usermod -u <uid> <user>
groupmod -g <gid> <user>
```

- Change ownership of your files to the new uid and gid. In this example I am assuming 1001 was the uid and gid assigned by default when creating `<user>` in the local machine, adapt them if needed:

```Bash
find / -group 1001 -exec chgrp -h <user> {} \;
find / -user 1001 -exec chown -h <user> {} \;
```

4. Reboot your server and login with your user.

### Install additional software

Once installation is complete, you will need to add the following packages:

1. `sudo apt update`
2. `sudo apt upgrade`
3. `sudo apt install tmux git git-flow vim vim-gtk zsh nfs-common cifs-utils filezilla docker.io openjdk-11-jdk openjdk-8-jdk net-tools desktop-file-utils openssh-server graphviz sshfs build-essential`
4. To install Google Chrome, download the .deb package and install it using (double click and install inside Ubuntu Software).
5. Optional software, according to your needs select the ones that apply to you.

```Bash
sudo apt install synaptic gnome-tweak-tool  gimp inkscape okular 
```

## Basic bioinformatics software

Install micromamba. Download latest miniconda bundle in [micromamba download page](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html).

## Mount internal disk at startup

1. Get the uuid of your disk

```
ls -al /dev/disk/by-uuid/
```

2. Copy the resultant UUID (for your disk) and then open fstab for editing

```
sudo vim /etc/fstab
```

3. Write the following lines at the end of the file by replacing the example value of the UUID by the one your disk

```
# data drive
UUID=19fa40a3-fd17-412f-9063-a29ca0e75f93 /media/data   ext4    rw,nosuid,nodev,relatime,stripe=8191 0 0
```

4. Test the fstab before rebooting (an incorrect fstab can render a disk unbootable).  To test do:

```
findmnt --verify
```

Check the last line for errors.  Warnings can help in improving your fstab.

## Set up BackUps plan

Install BackInTime:

```
sudo add-apt-repository ppa:bit-team/stable
sudo apt-get update
sudo apt-get install backintime-qt4
```

You can learn more about BackInTime at [documentation site](https://backintime.readthedocs.io/en/latest/index.html).

Backup schedule are running weekly, using BackInTime. to backup our home folder in a remote disk. We are mostly worried about our users data, software can always be reinstalled and containers redownloaded. In order to set up the backup of your home folder of your workstation, follow these steps:

1. Create your user in the machine containing the backup disks. At this point, the machine is `pc046614`.
2. Create a new rsa key and copy it to the repository:

```
ssh-keygen
ssh-copy-id <user>@10.15.60.49
```

3. ssh into `pc046614` and create your backup folder (if it does not exists already):

```
mkdir -p /media/backup
```

4. Open BackInTime in your machine and create the following backup plan:
![](https://raw.githubusercontent.com/BU-ISCIII/BU-ISCIII/master/images/backintime_general.png)
5. Select your home folder in the include tab, and whatever you want to exclude in the correspondent tab.
6. In autoremove, set it to this:
![](https://raw.githubusercontent.com/BU-ISCIII/BU-ISCIII/master/images/backintime_autoremove.png)
7. Click ok and done! Create your first one on demand to check if everything works.

## Keep ssh sessions alive

Asterix will kick you out every time you keep your connection idle for a set period of time. To avoid this annoying feature of the firewall, you can modify your ssh connection settings to send NULL packages every X seconds, which will keep your connection up and running.

How you do it, depends on the tool you are using for ssh-ing. I will explain the modifications you have make in the most broadly used tools to send a keep-alive signal every 5 minutes.

### SHH

Add the following lines to your "/etc/ssh/ssh_config" file `sudo nano /etc/ssh/ssh_config`

```
# Keep alive
Host *
    ServerAliveInterval 300
    ServerAliveCountMax 2
```

Now restart ssh service:

```
sudo service ssh restart
```

### PUTTY

In your session properties, go to Connection and under Sending of null packets to keep session active, set Seconds between keepalives to 300.

## Configure printers

The printer network in ISCIII is connected via samba, so first we need to install the samba client package:

```
sudo apt install smbclient
```

Also, you will need to download and extract the [PPD files for the printer models we have](https://cdn.kyostatics.net/dlc/es/driver/all/linux_8_1301_taskalfa3051.-downloadcenteritem-Single-File.downloadcenteritem.tmp/Linux_8.1301_TA...3051_x5x1ci.zip).

Now we can start configuring them:

1. Open ubuntu settings manager
2. Go to "Devices" -> Printers, scroll down to the bottom of the list of printers and click on "Additional Printer Settings"
3. Click on "Add", and in the new window select "Network Printer" -> "Windows Printer via SAMBA"
4. Set SMB Printer Address to:

```
smb://eunapio/IMVirtual
```

5. Set Authentication to "Set authorization details now" and fill the details with your domain password and isciii/<your_domain_username>
6. Click on "Forward" and now select the option to use your PPD file. Download the PPD file from the manufacturer, for Kyocera are [here](https://www.kyoceradocumentsolutions.co.za/index/service___support/download_center.false.driver.TASKALFA3501I._.EN.html#). You will need to select the right PPD file in "/wherever_you_extracted_PDP_folder/Linux_8.1301_TA...3051_x5x1ci/EU/Spanish/Kyocera_CS_3051ci.PPD".
7. Finish the configuration setting the alias of the printer if you want and you are ready to go!

#### WARNING

Due to internal configuration, if you are not allowed to print in colour you will not be able to print whatever you desire. You can only print word documents by opening them with libreoffice and printing with the advanced option "Colour: print text in black".

## Monitoring tools

Additional to htop you can install "btop+" tool to get more information about the process running, memory usage, and some other useful information about your workstation.

To install btop execute these steps

1. Install compiler version 10 and configure settings

```
sudo apt install coreutils sed gcc-10 g++-10

 # Select the g++-10 compiler instead of version 9
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 40
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 60
sudo update-alternatives --config g++
 # Select in the menu the option for g++-10
```

2. Create a temporary directory under Downloads folder

```
mkdir ~/Downloads/tmp
cd ~/Downloads/tmp
```

3. Clone the github respitory located at <https://github.com/aristocratos/btop>

```
git clone https://github.com/aristocratos/btop.git .
```

4. Edit the Makefile and search by -std=c++20’ and replace by -std=c++2a'

5. Then compile and install

```
sudo make
sudo make install
```

6. open the tool with "btop"
