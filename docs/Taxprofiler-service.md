# How to perform the Taxprofiler Service

## Introduction

This is a brief tutorial on how to perform a Taxprofiler Service as a member of the ISCIII's Bioinformatics Unit! Taxprofiler is a associated with _Taxonomic based Identification and classification of organisms in complex communities_ service from service catalog, but **usually, Taxprofiler is part of other services like [**IRMA**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/IRMA-service.md), [**viralrecon**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/Viralrecon-service.md), or [**PikaVirus**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/PikaVirus-service.md)**.

[**nf-core/taxprofiler**](https://github.com/nf-core/taxprofiler) is a bioinformatics best-practice analysis pipeline for taxonomic classification and profiling of shotgun short- and long-read metagenomic data. It allows for in-parallel taxonomic identification of reads or taxonomic abundance estimation with multiple classification and profiling tools against multiple databases, and produces standardised output tables for facilitating results comparison between different tools and databases. It is worth noting that this usage of the nf-core/taxprofiler pipeline does not allow for strain/subtype identification, so it only provides a general idea of the sample composition.

Let's get started with the service. When performing a taxprofiler service, it's likely that the service folder, resolution, and acronym are already created as part of these other services. Typical acronyms for a service that includes taxprofiler analysis are:

- `GENOME**` (for example: `GENOMEFLU`, `GENOMERSV`, `GENOMEEV`...)
- `VIRAL-DISCOVERY`
- `SARSCOV2`

**In the case that only the service _Taxonomic based Identification and classification of organisms in complex communities_ was requested in [iSkyLIMS](https://iskylims.isciii.es/), the most appropriate acronym is `VIRAL-DISCOVERY`. To perform this manual, we will assume that the service only contains Taxprofiler.**

## Service creation

>[!WARNING]
>**Please bear in mind this manual assumes taxprofiler is being done alone, but almost most of the times taxprofiler is done along with other services, so the service folder will probably be already created.**

First of all, take the service and click on _**Add resolution**_ in [iSkyLIMS](https://iskylims.isciii.es/) after logging in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym. In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/virology
ll -tr
```

Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered.

**Have a look at the last service that contains the word `VIRAL-DISCOVERY` in its name, and take note of the number linked to this term.**

For the new service, your service acronym will be this number + 1. For example, if the last folder after executing `ll -tr` is `SRVCNM1286_20250103_VIRAL-DISCOVERY028_miglesias_S`, the service acronym for your new service will be _**VIRAL-DISCOVERY029**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools micromamba environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/virology/
ll -tr
```

Now, let's execute the first BU-ISCIII tool: `new-service`, where you'll need to specify the resolution ID associated to this service.

```shell
buisciii new-service SRVCNMXXX.X
```

By default, **a `.log` file from this module's execution will be saved for tracking purposes in the service folder that will be created within `services_and_colaborations`**. This log file will have the following structure: `SRVCNMXXX.X.tool.log`, where `tool` is the name of the buisciii-tools module being launched. For instance, the log file will be named `SRVCNMXXX.X.new-service.log` if the module you are launching is `new-service`.

>[!NOTE]
>If you need the `.log` file to be saved in your PWD for any reason, or you want it to have a different name, use the option `--log-file` and indicate the name of your log file, for example:
>```
>buisciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X
>```

Once `new-service` is executed, you'll be asked:

* `Do you want to skip folder creation?`: Answer **NO** if the folder corresponding to the service has not yet been created in the `services_and_collaborations` folder. Otherwise, reply **YES**.
* Next, specify `taxprofiler`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_VIRAL-DISCOVERYXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:

* `lablog_taxprofiler`: an executable file that creates the `00-reads` folder which will contain all the reads for the analysis.
* `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

Let's execute the `lablog_taxprofiler` file:

```shell
bash lablog_taxprofiler
```

After executing this file, if everything is OK, we can now proceed with the new BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the analysis.

```shell
bu-isciii scratch SRVCNMXXX.X
```

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_VIRAL-DISCOVERYXXX_researcher_S/ANALYSIS/DATE_ANALYSIS01_TAXPROFILER
```

> [!WARNING]
> Please note that the Taxprofiler service is usually carried out along with other services. Therefore, please feel free to change the number associated with `ANALYSIS` accordingly. Remember to run all the scripts from other services before running the scripts from this service.

Once we're inside, we can execute our next executable file: `lablog`, which will create symbolic links to our raw reads and the `samples_id.txt` file. Besides, it will ask the user if they want the trimmed reads to be saved, apart from setting up all the necessary files that are needed for the execution of this nf-core pipeline via Nextflow.

```shell
bash lablog
```

Before continuing, please remember to load all the necessary modules:
```shell
module load Nextflow singularity
```

Besides, you should have the following files in the `DOC` folder. Please check you have them before running anything else:
* `taxprofiler.config`: a config file used by taxprofiler with the specific configuration for nf-core/taxprofiler in the HPC.
* `databases.csv`: a .csv file containing the absolute paths of all the databases that are employed by the classifiers launched by nf-core/taxprofiler.

After executing the `lablog` file, you should see the following folders/files:
* `samplesheet.csv`: a .csv file containing all the necessary information for taxprofiler to run. This file is used as input for the pipeline.
* `taxprofiler.sbatch`: an .sbatch file that will be used to run Nextflow on the nf-core/taxprofiler pipeline, using `samplesheet.csv` as input.
* `_01_nf_taxprofiler.sh`: an executable file that will launch taxprofiler.

Now, if we've loaded all the needed dependencies, we can start the analysis:

```bash
bash _01_nf_taxprofiler.sh
```

After this, the analysis will start, and we'll be able to check the status of the process with:

```bash
tail -f DATE_taxprofiler.log
```

Once checked everything has finished OK in the `DATE_taxprofiler.log`, within the `DATE_ANALYSIS0X_TAXPROFILER` folder, we'll have the following subfolders:

- `bowtie2`: Results from the host-read removal procedure carried out with BowTie2.
- `bracken`: results from post-processing procedure done with bracken from the results from kraken2.
- `centrifuge`: results from taxonomic classification done with Centrifuge.
- `fastp`: results from read pre-processing done with fastp.
- `fastqc`: results from quality control done with FastQC. 
- `kaiju`: results from taxonomic classification done with Kaiju.
- `kraken2`: results from taxonomic classification done with Kraken2.
- `krona`: results from krona after plotting the results obtained from the employed classifiers (kraken2, centrifuge, kaiju and bracken)
- `metaphlan`: results from taxonomic classification done with MetaPhlAn.
- `multiqc`: results after running MultiQC on the output of the other software run as part of this pipeline, generating an interactive .HTML report.
- `samtools`: results obtained after running samtools for the generation of several statistics of interest.

## `RESULTS` folder

Once the service is finished, go to `RESULTS` and execute the corresponding `lablog_taxprofiler_results`. After running this file, you should always check the following files and folders:

- `multiqc_report.html`: a MultiQC .HTML report containing all the relevant information obtained after running nf-core/taxprofiler, including quality control, reads pre-processing, host-reads removal, taxonomic classification after using several software, etc.
- `/krona`: within this subfolder, you should find four .HTML reports, one for each of the programmes that were used for taxonomic classification (kraken2, centrifuge, kaiju and bracken).

---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

    $ bu-isciii finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (in **Taxprofiler**, no files/folders are deleted). Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/virology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ bu-isciii bioinfo-doc SRVCNMXXX.X

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

>[!WARNING]
>When running the `delivery` mode of the `bioinfo_doc` module, you will be asked for **delivery notes** and **email notes**. **THESE ARE NOT THE SAME THING**. After running the `service_info` mode of this module, you'll see a folder for the service will have been created in `bioinfo_doc`. There, you can for example create two files: `delivery_notes.txt` and `email_notes.txt`. Edit these two files, and add the following information in each one of them:
>* `delivery_notes.txt`: `Results were delivered in the SFTP.` (literally)
>* `email_notes.txt`: everything you want the researcher to be aware of.

## Taxprofiler report template

>[!NOTE]
>Please remember this service is usually not done by itself, but along with other services like [IRMA](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/IRMA-service.md) or [viralrecon](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/Viralrecon-service.md), so it might be more appropriate to use the templates from these services instead of the following one, since it's specific for taxprofiler.

When everything is properly copied into the SFTP folder, you can put the update in the `update_servicios` channel's _Team Standup Bot_, with the following template:

- _**SRVCNMXXXXXX - ACRONYMXX**_:
  - **Servicio**: Taxonomic based Identification and classification of organisms in complex communities (taxprofiler)
  - **Número de muestras**: XXX
  - **Estado**: XXX
  - **Instrumento y longitud**: XXX *(example: NovaSeq (2x150))*
  - **Cantidad de lecturas**: XXX *(example: 0.8M - 91.2M)*
  - **Calidad general**: XXX
  - **Resultados generales**: brief summary of the results, after checking the .HTML reports from `RESULTS`.
  - **Incidencias generales**: indicate any anomalies you may have seen during the review of the results.
