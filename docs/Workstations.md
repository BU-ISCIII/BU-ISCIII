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

> **Note:** *For the moment, it is recommended installing [22.04.5 LTS ubuntu version](https://releases.ubuntu.com/22.04/?_ga=2.34887202.2033516692.1736508482-451044865.1736508482&_gl=1*qqnjfr*_gcl_au*MTc4NjEyOTkxNC4xNzM2NTA4NDgz) to prevent compatibility conflicts.*

In your Linux server open the "startup disk Creation" tool to create a bootable USB disk.

## Clean installation

1. Boot the server using the bootable disk.

When server is planned for computation propose, the installation settings are different to prevent for one side that critical partitions could full by user data and for other hand to install software packets for this propose.

Once your machine has been rebooted with the bootable disk plugged in, and the boot mode has been chosen, you will see the language selection menu followed by Ubuntu’s boot options. Then, before running the installation itself, click the "**Try Ubuntu**" button as shown in the image below.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/Tryubuntu.png)

As soon as the demo OS is loaded, open the Gparted software.
First, select the corresponding disk in the top right corner. If you are using a machine with the Linux operating system pre-installed, you will see something similar to this:

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_1diskselection.png)

In this case, you must select **/dev/nvme0n1** disk.

> **Note:** *The name of the disc may differ from the one shown in the image.*

The screen now will show the partitions of the selected disc.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_2partitions.png)

Since you are installing a native Ubuntu SO, you can delete all existing partitions. Note that doing this will erase all the information on the disk, so if you wish to preserve that data, consider making a backup beforehand.

Select and delete partitions. Then click the "apply all operations" button. You will see now all the free space unallocated.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_3erasedbeforeadd.png)

You can now create new partitions by clicking on the ‘New’ option in the newly freed space.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_5newpartition.png)

- Now, create 3 new partitions as follows:
    1. Create swap primary partition at the end of the disk (8000 MB). Select filesystem as ext4 and partition name and label as _swap_.

    <div style="text-align: center;">
      <img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_6swap.png" alt="image">
    </div>

    2. Create primary partition destined to the EFI (2000 MB). Select FAT32 as filesystem, and label it _efi_.

    <div style="text-align: center;">
      <img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_7efi.png" alt="image">
    </div>

    3. Create pvolume with the rest of disk space. Select as filesystem lvm2 pv, and as partition name and label pvolume.(Remaining space)

    <div style="text-align: center;">
      <img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_8pvolume.png" alt="image">
    </div>

Once the partitions have been created, apply the changes and then close Gparted.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_9partitionscreatedafteradding.png)


Now create vgroup and lvolumes with the command line as follows, where <vgroup> is "vg_<your_pc_name>", and <your_pc_name> is "PC" followed by the numeric code on the isciii's label of the physical machine (*e.g. vg_PC-XXXXX*). First of all, run the command "pvdisplay" with root privileges to get the name of your computer's physical volume (PV).

```Bash
sudo pvdisplay
```

Take the "PV Name" and substitute it in the next command:

```Bash
vgcreate <vgroup> <PV Name>
```

Finally, allocate space for the logical volumes.

```Bash
lvcreate -L 20G -n lv_root <vgroup>
lvcreate -L [whatever space you want to assign to your home]G -n lv_home <vgroup> # Left some space free so you can reassign this space in whatever partition you need in the future.
lvcreate -L 30G -n lv_opt <vgroup>
lvcreate -L 20G -n lv_tmp <vgroup>
lvcreate -L 20G -n lv_var <vgroup>
lvcreate -L 2G -n lv_var_log <vgroup>
lvcreate -L 20G -n lv_root_all <vgroup>
```

This is an example of PV creation from the command line.

<div style="text-align: center;">
  <img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_logicalvolumes.png" alt="image">
</div>

After completing all this configuration, proceed with the installation of the operating system by clicking on the icon in the lower right corner of the desktop.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ws_continueinstallation.png)

Then, in the installation GUI select the following options:

1. _Welcome_: Select `English` in the left panel and `Continue`
2. _Keyboard layout_: `Spanish` > `Spanish`
3. _Updates and other software_: Select:
    - `Minimal installation`
    - `Install third-party software`...
4. _Installation Type_: Select:
    - `Something else`
    - `Continue`

    Now you will assign every mounting point to the PV previously created. In the next window (Instalation type), select each volume one at time and then click de button `Change` for opening the pop-up window where the following settings will be applied:

    ```Bash
                Mount point   Use as
    efi         ->  -             (EFI System Partition)
    lv_root_all ->  /             (ext4)
    lv_root     ->  /root         (ext4)
    lv_home     ->  /home         (ext4)
    lv_opt      ->  /opt          (ext4)
    lv_tmp      ->  /tmp          (ext4)
    lv_var      ->  /var          (ext4)
    lv_var_log  ->  /var/log      (ext4)
    swap        ->  -             (swap area)
    ```

    - `Continue`

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
![](https://raw.githubusercontent.com/BU-ISCIII/BU-ISCIII/main/images/backintime_general.png)
5. Select your home folder in the include tab, and whatever you want to exclude in the correspondent tab.
6. In autoremove, set it to this:
![](https://raw.githubusercontent.com/BU-ISCIII/BU-ISCIII/main/images/backintime_autoremove.png)
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
