# Introduction

BU-ISCIII has two possible methods for results sharing with the requesting users.

- **SAMBA share** located in neptunovirtual machine (See more in [[Server-Bioinfo01]])

- **Bioinformatics sFTP resource** managed with syncplify.me software mantained by the IT department.

Here we describe how to use this resources.

## Cleaning instructions

Independently of the delivery method you **MUST Clean the service folder** before making the delivery to the user. Only necessary results are shared with the researcher, therefore you need to mark somehow the folders you don't want to copy. We use "_NC" mark to rename folder we are not going to copy, by default we rename:

- RAW to RAW_NC
- TMP to TMP_NC
- Inside analysis:
  - 00-reads to 00_reads_NC
  - 02-preprocessing to 02_preprocessing_NC

Depending on the analysis more folders may not be needed, please evaluate which files are needed specially when they are really BIG files like fastq, bam, etc (Send only one bam per sample). Moreover you need to assure that analysis has been **cleaned** and not duplicates or intermediate files are sent to the researcher (unless the researcher specifically has requested them).

## Delivery methods

### SAMBA shares

Neptuno is connected to ISCIII active directory and has a samba server for folder sharing. Configuration of new shared folders is delegated to user **bioinfoadm** managed by BU-ISCIII. Here we explain how to create new shared folders in neptuno:

1. Connect to neptuno using ssh, please ask for password if needed.

```Bash
ssh -X bioinfoadm@neptuno
```

2. Samba config file for shared folder configuration is located in ```/etc/samba/smbbioinfoadm.conf```. Each folder configuration is set using this template:

```shell
[bioinfo_doc]
   comment = Carpeta Compartida Unidad de Bioinformatica
   path = /srv/bioinformatica/bioinfo_doc
   browsable = yes
   read only = no
   guest ok = no
   valid users = @Bioinfo,iskylims
   #read list =
```

You can give access to active directory users, or to local neptuno users as in the example. For local users you have to create the local user and add it to the active directory database. For users in the active directory, just use their email without @isciii.es as username and, if it does not exists in the local machine, it will autmatically check in the active directory list.

```shell
## Look for commands to do this
```

3. Restart samba server for the changes to be effective.

```Bash
sudo systemclt restart smb
```

In order to deliver a service using SAMBA share you have to:

1. Create the samba share for the requesting lab if it does not exist yet.
2. Grant access to the requesting user, using its Active directory user.
3. Copy the results file to the shared folder.

```Bash
rsync -rlv {result_files} bioinfoadm@neptuno:/srv/bioinformatica/{shared_folder}
```

4. Send the email with the needed instructions for the user to download the results.

### sFTP

ISCIII's sFTP is managed using [syncplify.me](https://www.syncplify.me/) software. BU-ISCIII has access to its own virtual sftp site ```sftpbioinfo```. Instructions for accessing the resouce are located in [[bioinfo_doc]]/documentation/sftp/Guía de Administración para Bioinformática de su Virtual SFTP.pdf. Also in this folder you can find a easy tutorial for setting up Filezilla and access the sftp.

BU-ISCIII has a shared folder named sftp an located in ```/data/bi/sftp``` with 1 tb available for data sharing. Any folder/file copied to this directory is seen in the virtual sftp site for access and sharing configuration. A folder for each ISCIII research group is generated and shared with the service requesting users, also custom folders can be generated for specific purposes. sftp folder looks like this:

```Bash
pwd
    /data/bi/sftp
ls
labbacteriavac
SpainUDP
GeneticDiagnosis
...
```

#### Steps for delivering a service

Use bu-isciii tools delivery module. If the researcher does not have a user and folder configured in the sftp you will need to follow the [this manual](./SFTP-management.md).
