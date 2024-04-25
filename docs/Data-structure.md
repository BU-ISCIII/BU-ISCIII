# Introduction

In this document we'll describe how BU-ISCIII's filesystem works.

There are two main folders used by BU-ISCIII:

* **bi:** this folder is located in HPC's storage cabin (quibitka at the moment). We access it using sshfs since ssh is the only way to access HPC environment. **You need a HPC user to access, if you don't have so, please ask for it.** This folder is dedicated for bioinformatic analysis including, raw data (fastq), bam files, script and all possible results. This folder is not **backed up**, please be careful!

* **bioinfo_doc:** this folder is located in Panoramix VM inside [[Server Bioinfo01]] and is dedicated to BU-ISCIII documentation files. Therefore only **doc files** must be stored here, for example, service documentation, congress abtracts and files, training documentation,..

## Accessing files

In order to access them, first you will need an IT account with permissions to do it.

* For bioinfo_doc you need a Active directory user (windows/domain ISCIII user) and you need to be granted access permission. If you don't have it please ask for it.
* For data/bi you need a HPC user, if you don't have so, please ask for it.

## 1. Working on your local workstation/PC

By default, it is not likely the folders will be mounted on your local workstation/PC, unless another user manually mounted them. If that's the case, check who mounted the folders and under which user they are configured. You should unmount them if your user is not the one who mounted them, as you could experience errors (permissions, passwords, file ownership, …) if you use a different user.

By default we mount the folder into /data/ because that's the HPC organization, so we reproduce it in our workstations in order to make testing easier.
If you mount the remote folders in your local machine, you will need to create the folders “/data/bi/” and “/data/bi/” in your local machine (NOTE: you will need admin permissions to create them at “/”). Once they are created, mount the remote folders at need by entering the following commands in your terminal (changing <your_username> with your user name,<uid> with your user uid, and <gid> with bioinfo group gid - you can find this information in /etc/passwd file) :

```Bash
grep "<username>" /etc/passwd # for user info
grep "bioinfo" /etc/group # for group info
## You will see something like this bioinfo:x:1201:smonzon,mjuliam. The number 1201 is the gid or uid 
```

How to mount bioinfo_doc:

```Bash
mount -t cifs -o username=<your_domain_username>,domain=ISCIII,uid=<uid>,gid=<bioinfo_gid> //neptuno/bioinfo_doc /data/bioinfo_doc
```

How to mount bioinformatics:

```Bash
# ask for the port if you don't know it
sudo sshfs -p XXXXX -o allow_other,default_permissions <your_username>@portutatis.isciii.es:/data/bi /data/bi
```

If stuck, `kill -9` and:

```sudo fusermount -uz /data/bi```

Remember to unmount them when you finish working on them. Remote connections over the intranet timeout quickly, and leaving them mounted could cause you problems (usually fixed by hard unmounting the folders or rebooting your machine) next time you try to mount them. Sometimes this can be tedious and there is a way the connection won't die so ofter, see [[FAQs]] for how to fix this.

## 2. Working on the HPC environment

Asterix and each of the Obelix nodes have direct access to “/data/bi/”. There is no point in accessing “/data/bioinf_doc/” static files from here, so the connection is not available from the HPC.

Also take notice than ssh connection to HPC is unidirectional, I mean you can only access the HPC from your workstation, and you cannot access your workstation from the HPC. This means you have to copy date from your workstation to the HPC and **NOT** backwards.
