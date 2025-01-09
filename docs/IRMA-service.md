# How to perform services with IRMA

## Introduction

[**IRMA** (Iterative Refinement Meta-Assembler)](https://wonder.cdc.gov/amd/flu/irma/) is a software that was designed for the robust **assembly**, **variant calling**, and **phasing** of highly variable **RNA viruses**.

>[!NOTE]
>This software is our go-to when analysing **Influenza** and the **Respiratory Syncytial Virus (RSV)** (in the case of **RSV**, the analysis is usually done along with [**nf-core/viralrecon**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/Viralrecon-service.md), for genomic reconstruction, reference mapping and variant calling).

First of all, when talking about IRMA, we refer to:
* **INFLUENZA**: the Viral Flu: Influenza fragment reconstruction and variant detection service that researchers can select when requesting a service.
* **RSV**: the Viral: Genomic reconstruction, reference mapping and variant calling and/or *de novo* assembly service that researchers can select when requesting a service.

### Software used

As stated before, this service is done for the analysis of **RNA viruses**, mainly **Influenza** and **RSV**. For this, the following software is employed:

#### IRMA

[**IRMA**](https://wonder.cdc.gov/amd/flu/irma/), as indicated previously, robustly automates iteratively-refined assembly of NGS data using features designed for RNA viruses like influenza or RSV. Without the need to specify a reference beyond selecting a pre-built module, IRMA gathers reads and polishes assemblies for viral genomes using a modular, flexible strategy that improves NGS data analysis outcomes for rapidly evolving viruses, especially those with segmented genomes.

IRMA features include: (1) per segment insertion, deletion, single nucleotide variant, coverage, and full allele statistics in tabular form that are easy for database ingest or human readability; (2) amended consensus files with mixed base calls (customizable) and phasing distance matrices; (3) a read count table for primary and secondary data that makes it easier to infer if there might be mixed subtypes in the sample; (4) per segment figures for read counts, coverage diagrams with highlighted SNVs, and phasing heat maps; and (5) per segment VCF files and IGV-ready assembly files via Samtools.

**IRMA currently can be used for the analysis of Influenza, RSV and Ebola viruses.**

#### Taxprofiler

[**nf-core/taxprofiler**](https://nf-co.re/taxprofiler) is a bioinformatics best-practice analysis pipeline for **taxonomic classification** and **profiling** of shotgun short- and long-read **metagenomic data**. It allows for in-parallel taxonomic identification of reads or taxonomic abundance estimation with multiple classification and profiling tools against multiple databases, and produces standardised output tables for facilitating results comparison between different tools and databases.

The results from IRMA are complemented by the ones from nf-core/taxprofiler, in order to identify properly the organisms that are present in the samples being analysed, along with their corresponding subgroups, depending on the viral species.

## Bioinformatics procedure

>[!WARNING]
>As stated before, IRMA is usually run along with other pipelines. Regardless of the viral species, IRMA is usually followed by nf-core/taxprofiler. However, when analysing **RSV**, **IRMA** is usually preceded by **nf-core/viralrecon**. Please take this into account, since you might have already created the service folder with its corresponding acronym.

### Service creation

First of all, take the service and click on _**Add resolution**_ in [iSkyLIMS](https://iskylims.isciii.es/) after logging in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym. In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/virology
ll -tr
```

Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered.

**Have a look at the last service that contains the word `GENOMEFLU` or `GENOMERSV` in its name, and take note of the number linked to this term.**

For the new service, your service acronym will be this number + 1. For example, if the last folder after executing `ll -tr` is `SRVCNM1286_20250103_GENOMERSV028_miglesias_S`, the service acronym for your new service will be _**GENOMERSV029**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools micromamba environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/virology/
ll -tr
```

Now, let's execute the first BU-ISCIII tool: `new-service`, where you'll need to specify the resolution ID associated to this service.

```shell
bu-isciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X
```

The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results).  

Once `new-service` is executed, you'll be asked:

* `Do you want to skip folder creation?`: Answer **NO** if the folder corresponding to the service has not yet been created in the `services_and_collaborations` folder. Otherwise, reply **YES**.
* Next, specify `IRMA`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_GENOMEXXXXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:

* `lablog_irma`: an executable file that creates the `00-reads` folder which will contain all the reads for the analysis.
* `samples_id.txt`: a `.txt` file, obtained after running `lablog_irma`, containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

Let's execute the `lablog_irma` file:

```shell
bash lablog_irma
```

After executing this file, if everything is OK, we can now proceed with the new BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the analysis.

```shell
bu-isciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
```

Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_GENOMEXXXXXX_researcher_S/ANALYSIS/DATE_ANALYSIS01_IRMA
```

> [!WARNING]
> Please note that the IRMA service, in the case of RSV, is usually preceded by [viralrecon](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/Viralrecon-service.md). Therefore, please feel free to change the number associated with `ANALYSIS` accordingly.

Once we're inside, we can execute our next executable file: `lablog`, which will create symbolic links to our raw reads and the `samples_id.txt` file.

```shell
bash lablog
```

Before continuing, please remember to load all the necessary modules:
```shell
module load singularity R
```

Once we have reached this point, we can continue with the next step.

>[!WARNING]
>Bear in mind that IRMA does not include quality trimming of the raw reads by itself. Therefore, before the execution of the pipeline, the samples are analysed with [**FastQC**](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) for quality control and pre-processed with [**Fastp**](https://github.com/OpenGene/fastp?tab=readme-ov-file#fastp), as eill be explained below.
>
>**If you have already pre-processed your reads, the quality control and pre-processing steps do not have to be done again. You can go directly to the analysis with IRMA, taking into account that you'll have to manually modify the corresponding `lablog` to use the pre-processed reads that were already created.**

### Quality Control (FastQC)

- Move to the first folder: `cd *ANALYSIS01_IRMA/01-preproQC/`.
- Read the `lablog` file you'll find and execute it `bash lablog`. This will create an auxiliar script called `_01_rawfastqc.sh` and a `logs/` folder.
- Now you are ready to execute `_01_rawfastqc.sh`.

>[!NOTE]
[**FastQC**](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) is the software used in this step for quality control of the raw reads.

### Pre-processing (fastp)

- You can go on with this step while the first one is still running. Start with `cd 02-preprocessing`.
- Read the `lablog` carefully to check that everything is correct. Then, execute it `bash lablog`.
- Similar to the previous step, you will see a new `logs/` folder and an auxiliar script called `_01_fastp.sh`.
- Run `_01_fastp.sh` to execute [Fastp](https://github.com/OpenGene/fastp?tab=readme-ov-file#fastp).

>[!NOTE]
[**Fastp**](https://github.com/OpenGene/fastp) is the software used in this step for raw reads pre-processing.

### Post-processing Quality Control

- Now, move to `03-procQC/`
- **This is exactly the same as step 1**, but we will do a quality assessment of the reads previously filtered with fastp.

### IRMA
>[!WARNING]
>**In `04-irma/`, you'll find several files, each one with a suffix: flu or rsv. Taking this into account, run only the scripts that you need to run depending on the organism you are analysing.**
- In `04-irma/` you may find two files: `lablog` and `create_irma_stats.sh`. You should read them carefully before continuing with the analysis.
- Execute the `lablog` with `bash lablog`, this will create several scripts `_01_irma.sh` `_02_create_stats.sh` and `_03_post_processing.sh` (each script with its corresponding suffix), along with a new `logs/` folder.
- Execute the scripts sequentially.

### Taxprofiler

- [**Taxprofiler**](https://github.com/nf-core/taxprofiler) is used in this workflow to perform taxonomic classification of the reads and detect sources of contamination in the samples. You can read the [**Taxprofiler documentation**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/Taxprofiler-service.md) for further description on the methodology of this analysis.

## `RESULTS` folder.

Once the service is finished, go to `RESULTS/` and execute the corresponding `lablog_irma_results`. Depending on the type of Influenza Virus or RSV found in the samples, you may find different files, but you should always check the following:

- `taxprofiler/` is a folder that includes the results obtained after running taxprofiler, consisting in a MultiQC .html report and all the .html reports obtained after running krona on the report from all the classifiers that are run by means of taxprofiler.
- `all_samples_completo.txt` includes the .fasta sequence for all the fragments in all the samples.
- Depending on the virus types found in your samples, you will have one folder for each fragment found (e.g. A_H1 for Influenza A H1_N1).
- In case of Influenza services (FLU), you will find a file named `flu_type_summary.txt`, which will include a summary of the different types of influenza found in your samples. This is the most relevant information to be included in your report. You'll also find a similar file for RSV services, called `rsv_type_summary.txt`, indicating the detected RSV subgroups for your samples.

---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

    $ bu-isciii --log-file SRVCNMXXX.X.finish.log finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (in **Snippy**, this folder is `02-preprocessing`). Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ bu-isciii --log-file SRVCNMXXX.X.tool.log bioinfo-doc SRVCNMXXX.X

Remember to save the logs with the corresponding name (i.e. `SRVCNMXXX.X.service_info.log` or `SRVCNMXXX.X.delivery.log`).

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

## IRMA report template

This is just a general template, but bear in mind you might have to add information from viralrecon, if you ran it before IRMA.

***SRVCNMXXX_XXXXXXXX_GENOMEXXXXX_XXXXXXXXX***
* **Servicios**: (Viralrecon), IRMA y Taxprofiler.
* **Referencias**: _Indicate the ID of the reference/s that was/were employed for the analysis_.
* **Número de muestras**: XXX
* **Estado**: XXX
* **Instrumento y longitud**: XXXXX (XxXXX)
* **Cantidad de lecturas**: XXXXX
* **Calidad general**: XXXXX
* **Resultados generales**: _Typically_, you have to indicate the results from the summary.txt you generated in the `RESULTS` folder_
* **Incidencias muestras individuales**: *indicate any incidences that you might consider relevant for the researcher, either coming after running viralrecon, IRMA or taxprofiler*.
---