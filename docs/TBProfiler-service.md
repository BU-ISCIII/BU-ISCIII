# How to perform the TBProfiler Service

## Introduction

This is a brief tutorial on how to perform a TBProfiler Service as a member of the ISCIII's Bioinformatics Unit! TBProfiler is a associated with *Bacteria: In-depth analysis of Mycobacterium species genomes (e.g M. tubercolis, M. bovis)* from the service catalog.

[**TBProfiler**](https://github.com/jodyphelan/TBProfiler) is a computational pipeline designed for the analysis of **Mycobacterium tuberculosis (MTB) whole genome sequencing (WGS) data**. The pipeline aligns reads to the H37Rv reference using [bowtie2](https://github.com/BenLangmead/bowtie2), [BWA](https://github.com/lh3/bwa) or [minimap2](https://github.com/lh3/minimap2) and then calls variants using [bcftools](https://github.com/samtools/bcftools). These variants are then compared to a [drug-resistance database](https://github.com/jodyphelan/tbdb) and annotated using [SnpEff](https://pcingola.github.io/SnpEff/). It also predicts the number of reads supporting drug resistance variants as an insight into hetero-resistance.

# Service creation

First of all, take the service and click on _**Add resolution**_ in [**iSkyLIMS**](https://iskylims.isciii.es/) after logging in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym. In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/bacteriology
ll -tr
```

Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered. Have a look at the last service that contains the word `MTUBERCULOSIS` in its name, and take note of the number linked to this `MTUBERCULOSIS` term.

For the new service, your service acronym will be this number + 1. For example, if the last folder after executing `ll -tr` is `SRVCNM1247_20241106_MTUBERCULOSIS75_lherrera_S`, the service acronym for your new service will be **MTUBERCULOSIS76**. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools conda environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/bacteriology/
ll -tr
```

Now, let's execute the first BU-ISCIII tool: `new-service`, where you'll need to specify the resolution ID associated to this service.

```shell
buisciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X
```

The option `--log-file` will save a .log file for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results).  

Once `new-service` is executed, you'll be asked:

* `Do you want to skip folder creation?`: Unless it is not the first resolution associated with the service, answer **NO**, because the folder corresponding to the service has not yet been created in the `services_and_collaborations` folder. However, please consider that **this TBProfiler service is usually performed along with the assembly service**, so usually the service folder will have already been created.
* Next, specify `tbprofiler`, since this is the service we're running (you'll already have indicated `assembly_annot` previously when running `new-service` for the first time).

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_MTUBERCULOSISXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:

* `lablog_tbprofiler`: an executable file that creates the `00-reads` folder which will contain all the reads.
* `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

Let's execute the `lablog_tbprofiler` file:

```shell
bash lablog_tbprofiler
```

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the TBProfiler analysis.

```shell
buisciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
```

Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_MTUBERCULOSISXXX_researcher_S/ANALYSIS/DATE_ANALYSIS02_TBPROFILER
```

> [!WARNING]
> Please note that the TBProfiler service is usually performed along with the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service); that's why this folder is called `ANALYSIS02`. During this assembly service, **we should have saved the trimmed sequences**, since **they will be needed** during the TBProfiler analysis.

Once we're there, and before executing anything else, we should load all the necessary dependencies:

```shell
module load singularity
```

Once we're inside this folder, we'll find a `lablog` file. Please execute this lablog, so the following files are created:

* `00-reads`: symbolic links to the fastq.gz files.
* `samples_id.txt`: text file containing the samples ids.
* `_01_tbprofiler.sh`: bash script containing TBProfiler pipeline execution commands for each sample.
* `_02_tbcollate.sh`: bash script that merges TBProfiler analysis output from *_01_tbprofiler.sh*.

> Beforehand, please check tha symlinks and samples_id.txt is properly created with the corresponding samples for the requested analysis.

Firstly, **execute the *_01_tbprofiler.sh*** file to launch the `tbprofiler profile` pipeline:

```shell
bash _01_tbprofiler.sh
```

One job will be sumbitted per sample and logs for each will be saved in `logs/` folder as `TBPROFILER.SAMPLE.JOB_ID.log`.

This pipeline will generate three output folders:

* `bam`: BAM and BAI files for each sample.
* `vcf`: VCF.gz files for each sample.
* `results`: TBProfiler results for each sample in csv and json format.

Once all the jobs are completed, you can proceed to execute the second script:

```shell
bash _02_tbcollate.sh
```

This script will merge the results from all samples and generate three files:

* `tbprofiler.txt`: summary table with the main results for each sample (lineage, drug resistance, mapped reads, etc.).
* `tbprofiler.variants.csv`: table with all the drug resistance variants found in each sample (affected gene, position, mutation, etc.).
* `tbprofiler.variants.txt`: matrix table with the presence/absence of each variant in each sample.

Once all scripts, from the **Assembly** and the **TBProfiler** services, have been run, we can go to the `RESULTS` folder of the service, and we should have the scripts `lablog_assembly_results` and `lablog_tbprofiler_results` there. The first lablog is already described in the [Assembly service](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) wiki page, while the second one creates, within the `DATE_entrega01` folder, a subfolder called `TBProfiler`, inside which we'll find symbolic links to the merged tables generated by the *_02_tbcollate.sh* script. The files that should be linked within this `TBProfiler` folder are the above mentioned:

* `tbprofiler.txt`
* `tbprofiler.variants.csv`
* `tbprofiler.variants.txt`

---

### What should I pay attention to before the delivery of the service?

Before we deliver the service, we should check the files that we have inside the `RESULTS` folder. In the case of the TBProfiler service, we should check the that the merged tables are properly generated and linked within the `TBProfiler` folder. In particular, we should check the following:

* `tbprofiler.txt`: check that a .txt file with this name is stored for every sample within this folder. This file contains the main results of the TBProfiler analysis for each sample, including the lineage classification and the drug resistance prediction. Especially check the following columns:
  * `main_lineage`: check if the lineage assigned to each sample match the one indicated by Kmerfinder.
  * `pct_mapped_reads`: check that the number of mapped reads is reasonable (i.e. not too low, which could indicate a problem with the data quality or with the analysis).
  * `Drug resistance`: check which drugs are predicted to be resistant for each sample, and check if this matches with the `tbprofiler.variants.csv` file (i.e. if a sample is predicted to be resistant to rifampicin, check that there are indeed variants in the rpoB gene that are linked to rifampicin resistance).
---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools.

    $ buisciii --log-file SRVCNMXXX.X.finish.log finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space. Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ buisciii --log-file SRVCNMXXX.X.tool.log bioinfo-doc SRVCNMXXX.X

Remember to save the logs with the corresponding name (i.e. `SRVCNMXXX.X.service_info.log` or `SRVCNMXXX.X.delivery.log`).

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution. **Please indicate any incidences you might have found during the analysis in these notes**.

Lastly, remember to remove all the files related to this service from `scratch_tmp`:

    $ buisciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
    $ remove_scratch

---

## TBProfiler report template

**_SRVCNMXXXX - SERVICE ALIAS_**:

* **Servicios**: Ensamblado de novo, TBProfiler *(y análisis de SNPs)*.
* **Carrera**: run name.
* **Número de muestras**: XXXX
* **Solicitante**: XXXX
* **Laboratorio**: XXXX
* **Organismo indicado**: *M. tuberculosis*
* **Estado**: Finalizado y copiado al SFTP.
* **Cantidad de lecturas**: XXXX - XXXX
* **Calidad general**: XXXX
* **Especie identificada**: M. XXXX
* **Resultados generales**:
  * **Assembly**: indicate any information about the assembly procedure that might be relevant.
  * **TBProfiler**:
    * **Stats**: %mapped reads: XXXX.
    * **Classification**: Check whether the classification of the samples by TBProfiler matches the one provided by Kmerfinder.
* **Incidencias muestras individuales**: indicate any incidences that might be relevant, like samples that could not be assembled or samples for which the lineage could not be determined.

---

