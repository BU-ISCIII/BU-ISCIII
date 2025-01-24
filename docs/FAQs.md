# Index

- [Index](#index)
  - [How can I copy data from my Workstation to the HPC?](#how-can-i-copy-data-from-my-workstation-to-the-hpc)
  - [I can't copy data from the HPC to my workstation](#i-cant-copy-data-from-the-hpc-to-my-workstation)
  - [What if a certain run is not stored in `srv/fastq_repo` anymore?](#what-if-a-certain-run-is-not-stored-in-srvfastq_repo-anymore)
  - [What if the HPC is not working or it is working very slowly?](#what-if-the-hpc-is-not-working-or-it-is-working-very-slowly)
  - [How to mount `/data/ucct/bi/` and `/data/ucct/bioinfo_doc/`?](#how-to-mount-dataucctbi-and-dataucctbioinfo_doc)
  - [What if `/data/ucct/bi/` and `/data/ucct/bioinfo_doc/` unmount for any reason?](#what-if-dataucctbi-and-dataucctbioinfo_doc-unmount-for-any-reason)
  - [How can I install a new release of the BU-ISCIII tools in the HPC?](#how-can-i-install-a-new-release-of-the-bu-isciii-tools-in-the-hpc)
  - [How can I install a new version of a Singularity image in the HPC?](#how-can-i-install-a-new-version-of-a-singularity-image-in-the-hpc)
  - [How can I install a new version of an nf-core pipeline in the HPC?](#how-can-i-install-a-new-version-of-an-nf-core-pipeline-in-the-hpc)
  - [What if the researcher asked for the analysis of samples that do not appear on iSkyLIMS?](#what-if-the-researcher-asked-for-the-analysis-of-samples-that-do-not-appear-on-iskylims)
  - [What if the researcher selected the wrong services when submitting a request via iSkyLIMS?](#what-if-the-researcher-selected-the-wrong-services-when-submitting-a-request-via-iskylims)
  - [What if a sample appears on iSkyLIMS but it is not present inside `/srv/fastq_repo/`?](#what-if-a-sample-appears-on-iskylims-but-it-is-not-present-inside-srvfastq_repo)
  - [How to upload raw sequences into SRA](#how-to-upload-raw-sequences-into-sra)
  - [Decent working environment in Windows 10 WSL](#decent-working-environment-in-windows-10-wsl)
  - [Copy and export only visible cells in LibreOffice Calc](#copy-and-export-only-visible-cells-in-libreoffice-calc)
  - [I've installed docker and now I can't connect to my machine with ssh](#ive-installed-docker-and-now-i-cant-connect-to-my-machine-with-ssh)
  - [Boot partition is full?](#boot-partition-is-full)
    - [List installed packages that match a linux-image pattern](#list-installed-packages-that-match-a-linux-image-pattern)
    - [Identify the kernel that your machine is running](#identify-the-kernel-that-your-machine-is-running)
    - [Delete old kernels](#delete-old-kernels)

## How can I copy data from my Workstation to the HPC?

Run this command, substituting `<ws_path>` by the proper path of what you want to copy from the WS (`<ws_path>` is the origin) and `<destination_path>` by the proper path of the HPC where you want to copy the data of interest. Replace `user` by your HPC user as well:

```
rsync -rlv -aP -L --rsh='ssh -p32122' <ws_path> user@portutatis.isciii.es:<destination_path>
```

## I can't copy data from the HPC to my workstation

**Take notice that ssh connection to HPC is unidirectional**, I mean **you can only access the HPC from your workstation, and you cannot access your workstation from the HPC**. This means you have to copy date from your workstation to the HPC and **NOT** backwards.

## What if a certain run is not stored in `srv/fastq_repo` anymore?

**`srv/fastq_repo` is the directory where all raw data from the different runs that have been done in the past 1 year is stored**. This directory is essential since all services carried out within BU-ISCIII rely on it, because the `RAW` folder of every service contains symbolic links to these raw `.fastq.gz` files that are contained within their corresponding run folder in `/srv/fastq_repo`.

If a specific run is not recent, it might have already been archived and not appear in `/srv/fastq_repo`. If this happens to you, you'll have to follow these steps:

1. With your ISCIII credentials, go to [**sau.isciii.es**](https://sau.isciii.es/) and log in (you have to be physically in ISCIII or be connected to ISCIII's VPN to be able to do this).
2. Click on **+ Crear una petición**.
3. Fill in the form:
   
![Form](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/Form.png)
   * **Tipo**: Solicitud
   * **Categoría**: >> Sistemas, Comunicaciones y Seguridad >> **Clúster HPC**.
   * **Urgencia**: depending on the situation.
   * **Observadores**: add the people that should be monitoring this issue.
   * **Título**: something like "**Re-compartición de datos de secuenciación masiva**".
   * **Descripción**: something like:
  ```
  Querría recuperar el siguiente proyecto de secuenciación, del usuario XXXXXX, con el fin de que esté disponible en el recurso activo fastq_repo:

  XXXXXX_GEN_XXX_XXXXXXXX_XXXXXXXXX
  
  Muchas gracias de antemano.
  ```
4. Click on **+ Enviar mensaje**.
5. Keep an eye on your inbox, since you'll receive a reply on your request via email.
   
>[!NOTE]
>Please take into account that the previous approach is for when a run is older than 1 year. If the run that you cannot find is quite recent and you still cannot find it in `/srv/fastq_repo`, you should check iSkyLIMS to see if it was loaded correctly. If this is the case, please check **`NGS_Data`**, since the raw reads might be there.
>
>You must have NGS_Data mounted in your local PC. To mount it, run the following command (**you might have to ask for all pertinent permissions before mountint this folder, by submitting a request in sau.isciii.es**):
>```
>sudo mount -v -t cifs -o username=<user>,domain=ISCIII //galera.isciii.es/NGS_Data /data/NGS_Data
>```

## What if the HPC is not working or it is working very slowly?

There may be occasions in which you cannot log in the HPC or it works very slowly. In these cases, you should talk to other BU-ISCIII members in order to check whether this is also happening to them. You can also check [**Ganglia**](http://ganglia.isciii.es/?r=hour&cs=&ce=&m=load_one&s=by+name&c=XTutatis&tab=m&vn=&hide-hf=false) to see if any of the nodes or portutatis itself is down.

If you or any of the members of the Unit believe that the SAU should be aware of this, please submit a petition on [**sau.isciii.es**](https://sau.isciii.es/).

## How to mount `/data/ucct/bi/` and `/data/ucct/bioinfo_doc/`?

The **`/data/ucct/bi/`** resource is essential for the correct usage of the BU-ISCIII tools, since everything that is needed for the proper development of the services that are requested via [**iSkyLIMS**](https://iskylims.isciii.es/) is stored in `/data/ucct/bi/`.

Furthermore, all the information regarding the delivery of previous services, among other relevant stuff for the Unit, is stored in **`/data/ucct/bioinfo_doc/`**. Having access to this folder is also fundamental to be able to deliver finished services.

Once you are assigned a WS and a local user, you shouldn't have access to /data/ucct/bi/ and /data/ucct/bioinfo_doc/. You'll have to mount these folders in your WS. To do so, follow these steps:

1. Open a new terminal and lauch these commands. You'll be creating the `/data/ucct/bi/` and `/data/ucct/bioinfo_doc/` folders locally:
```
sudo mkdir -p /data/ucct/bi/
sudo mkdir /data/ucct/bioinfo_doc/
```
2. Mount `/data/ucct/bi/` in your WS
```
sudo sshfs -o reconnect,ServerAliveInterval=15,ServerAliveCountMax=3,allow_other,default_permissions -p 32122 <user>@portutatis.isciii.es:/data/ucct/bi /data/ucct/bi/
```
3. Now, mount `/data/ucct/bioinfo_doc/` in your WS
```
sudo mount -t cifs -o username=<user>,domain=ISCIII,uid=XXXX,gid=XXXX //neptuno.isciii.es/bioinfo_doc /data/ucct/bioinfo_doc
```

>[!NOTE]
> The username you have to indicate is the one from the HPC, not the one from your WS. You'll be asked to type your WS password and your ISCIII password (the one you use to log in your email, [SAU](sau.isciii.es), [iTramita](itramita.isciii.es), etc.)

>[!NOTE]
> In order to know your **UID** and **GID**, execute this command in your local terminal:
> ```
> id
> ```
> You'll see two variables, called uid and gid, which will have associated a number. This number is the one you have to indicate when mounting `/data/ucct/bi/` and `/data/ucct/bioinfo_doc/`.

## What if `/data/ucct/bi/` and `/data/ucct/bioinfo_doc/` unmount for any reason?

It is pretty common that these resources unmount for some reason, mainly because the power shut down at some point. In these cases, you'll have to mount them again running the commands from [How to mount `/data/ucct/bi/` and `/data/ucct/bioinfo_doc/`?](#how-to-mount-databi-and-databioinfo_doc).

If you need to unmount manually any of these folders (maybe because you mounted them wrong), run the following (notice this is for `/data/ucct/bi/`, but it can be done also for `/data/ucct/bioinfo_doc/`):

```
sudo fusermount -uz /data/ucct/bi
```

## How can I install a new release of the BU-ISCIII tools in the HPC?

If there is a new release of the BU-ISCIII tools, you should install it in the HPC. To do so, follow these steps:

1. Log in the HPC with your credentials.
2. Go to `/data/ucct/bi/pipelines/buisciii-tools/`.
3. Run `git branch` to check the branch in which you are right now. You should be in the `main` branch.
4. Create a new micromamba environment for the new release: `micromamba create -n buisciii-tools_X.X.X`.
5. Activate this new micromamba environment: `micromamba activate buisciii-tools_X.X.X`.
6. Run `git pull` to get the latest changes on the clone of the buisciii-tools GitHub repository.
7. While being in `/data/ucct/bi/pipelines/buisciii-tools/`, run `pip install .`

>[!NOTE]
>You might need to install `pip` in your new micromamba environment. To do so, run `micromamba install pip`.

## How can I install a new version of a Singularity image in the HPC?

Most of the programmes on which our templates rely are executed based on **Singularity images** (please find detailed documentation on Singularity [**here**](https://docs.sylabs.io/guides/3.0/user-guide/quick_start.html)). It is **recommendable** to keep the versions of these images **updated** regularly, but **always testing first that they work correctly before adding them into our templates**. 

What should be done in order to download a Singularity image? Follow these steps:

1. First, go to: **https://depot.galaxyproject.org/singularity/**.
2. In this page, you'll see several Singularity images ready to be downloaded. Pay attention to the names of the files and their submission dates (second column).
3. For example, let's say we want to download the latest mtbseq Singularity image. Press `Ctrl + F` to search for "mtbseq", and you'll see in yellow all the found occurrences. Once this has been done, simply check which image is the most recent.
4. Once identified, place your mouse on the most recent image and right-click on it. Click on "Copy link address".
5. Now, log into the HPC and go to `/data/ucct/bi/pipelines/singularity-images`. Here, run `wget` and paste into the terminal the link you just copied. Press Enter and your Singularity image will be downloaded inside this folder.

Once you have the new Singularity image, please test it before implementing it into the templates manually.

## How can I install a new version of an nf-core pipeline in the HPC?

If you need to install on the HPC a new version of an **nf-core pipeline**, this procedure should be done by means of the [**nf-core-tools**](https://nf-co.re/docs/nf-core-tools/pipelines/download). To do so, follow these steps:

1. Activate the corresponding micromamba environment: `micromamba activate nf-core-2.14.0`.
2. Go to the folder in which you want to download the new version of this pipeline. Let's say you want to download a new version of [nf-core-bacass](https://nf-co.re/bacass/2.4.0/): `cd /data/ucct/bi/pipelines/nf-core-bacass`.
3. Make sure you have added this line in your `.bashrc` file: `export NXF_SINGULARITY_CACHEDIR="/data/ucct/bi/pipelines/singularity-images"`, as stated in the [**Usage**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Usage) page in the wiki.
4. In your HPC home, create the following file:
```
cd ~
find $NXF_SINGULARITY_CACHEDIR -name "*.img" > my_list_of_remotely_available_images.txt
```
This is done so as to avoid unnecessary container image downloads. The .txt file created will contain a list of already available images as plain text, in this case the images stored in `/data/ucct/bi/pipelines/singularity-images/`.

5. Run the following command, which consists in three steps:
   * Inside the `/data/ucct/bi/pipelines/your-pipeline/` folder, create a new subfolder for the new version of the pipeline.
   * Go to this new subfolder.
   * Run the **`download`** module, specifying the following options:
     * `-r`: pipeline release to download.
     * `--container-cache-utilisation`: by indicating remote, you can avoid unnecessary container image downloads.
     * `--container-cache-index`: with this option, and indicating the route to the .txt file that indicates the images that are already downloaded, you can avoid unnecessary image downloads, as stated before.
     * `-x`: archive compression type.
```
mkdir nf-core-bacass-2.4.0 && cd nf-core-bacass-2.4.0 && nf-core download bacass -r 2.4.0 --container-cache-utilisation remote --container-cache-index ~/my_list_of_remotely_available_images.txt -x none
```

6. Check what you see on the screen, in order to make sure the process finishes without any errors.

## What if the researcher asked for the analysis of samples that do not appear on iSkyLIMS?

In some cases, a researcher might ask for the analysis of samples that do not appear on iSkyLIMS. There is not a universal solution for this, unfortunately, since this can be solved in different ways depending on the situation:
* For example, the researcher, in the request message, might directly give you a link from which you have to download the raw sequences. These files might not have been uploaded in iSkyLIMS. In this case, you'll have to download these sequences manually and insert them in the `RAW` folder of the service.
* Sometimes, the researcher may have named the samples in iSkyLIMS in one specific way, when the raw files are named after another structure. For example, the sample name in iSkyLIMS might be 680-24, but in reality the sample name is 20240680. Plus, the researcher might have indicated the wrong run/project. This depends very much of the situation, but it is recommendable to do some research on iSkyLIMS regarding the most recent runs that have been uploaded into this platform, in order to try to see if there are recent samples that could be the sample that we are really looking for.

## What if the researcher selected the wrong services when submitting a request via iSkyLIMS?

As you may know already, when preparing a request submission, the researcher has to select via [**iSkyLIMS**](https://iskylims.isciii.es/drylab/sequencing-request) which services they want us to carry out with the samples they indicate. 

It is not rare to notice that a researcher has selected the wrong service/s when submitting this request (or they have forgotten to add a service that is needed for the completion), and this will make `bu-isciii new-service` to offer the user to import templates that have nothing to do with the service requested. For example, imagine a researcher wants us to perform plasmid analysis (which corresponds to the Bacteria: Plasmid analysis and characterization service), but they have only selected the Bacteria: Multi-Locus Sequence Typing (MLST), analysis of virulence factors, antimicrobial resistance, and plasmids characterization service, which corresponds to characterization. In this case, we would need to add manually the service that is missing.

How can we do this? We should go in this situation to: **https://iskylims.isciii.es/admin/**. To access this page, you have to log in with the following credentials:
* **User**: bioinfoadm
* **Password**: IYKYK

After that, you'll get into a **Django admin interface**. You'll find several sections, including "**Authentication and authorization**", "**Core**", "**Django_utils**", "**Drylab**" and "**Wetlab**". In this case, we should:
1. Go to the **Services** section from **Drylab**, and there we'll see all the services that have been requested so far.
2. Click on the service ID for which this incidence has happened.
3. Once inside, go to the **AvailableServices** box and, while carefully pressing `Ctrl`, remove and include those services that you need.
4. Press **SAVE** at the end of the page, and your changes will have been saved.

Now, if you refresh the service page on iSkyLIMS, you'd see the correct services associated with your service. You can then add a new resolution for this service, accept it and start with the analyses :)

## What if a sample appears on iSkyLIMS but it is not present inside `/srv/fastq_repo/`?

You might come across a certain sample that appears on iSkyLIMS but which, when running `buisciii new-service`, cannot be found in `/srv/fastq_repo/`, as indicated in an error message like this one:
```
This regex has not output any file: /srv/fastq_repo/XXXXXXX_GEN_XXX_XXXXXXXX_XXXXXXX/XXXXXXXXX_*. This
maybe because the project is not yet in the fastq repoor because some of the samples are not in the 
project.
 Service has 39 number of selected samples in iSkyLIMS
 38 number of samples where found to create symbolic links
```
If you see an error message like the previous one, it means there is one sample for which there are no raw sequences in /srv/fastq_repo/. Check which sample is missing and search for it in iSkyLIMS, by accessing its corresponding run. You'll see something that will catch your attention. Can you guess what it is?

<details>
  <summary><b>Reveal solution</b></summary>
  
  You'll see there are no reads associated with the sample that is missing, and that the quality is 0. Therefore, this sample has not been sequenced and has no reads. You'll have to indicate this to the researcher when delivering the service results.

  **Example**:
  ![No-reads-sample](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/No-reads-sample.png)

</details>

## How to upload raw sequences into SRA

Please follow the [**SRA submission guide**](https://docs.google.com/document/d/11niGgcxZiYugji9nlNMvsMsgj6a-8efESJkO6P44WEk/edit?usp=drive_link).

## Decent working environment in Windows 10 WSL

You have [**here**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Decent-working-environment-in-Windows-10-WSL) a great tutorial for that.

## Copy and export only visible cells in LibreOffice Calc

Filtered and hidden cells are always copied with LibreOffice Calc. In order to copy only selected cells, you need to install an addon: [Copy only visible cells](https://extensions.libreoffice.org/extensions/copy-only-visible-cells).

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

### List installed packages that match a linux-image pattern

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

This [**website**](https://linuxprograms.wordpress.com/2010/05/11/status-dpkg-list/) explains how to interpret the two-code letters in the first column.

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
