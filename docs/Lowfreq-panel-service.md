# Low frequency variants detection and annotation in a sequencing panel

## Introduction

This service is requested by the Human Genetics staff at IIER, generally requesting the **annotation of low-frequency variants for the RB1 gene** (related to retinoblastoma) given a custom panel. This service is associated with the _Low-frequency variants detection and annotation for whole genome or sequencing panel (e.g. retinoblastoma gene panel)_ service from the service catalog.

Here's an example of request:
```
Análisis de variantes de un panel custom IDT del gen RB1 completo con exones, intrones y región promotora. Anotación de las variantes en los casos secuenciados. Incluir variantes en baja frecuencia (hasta un 1%).
```

### Software used

- **Read pre-processing**: low-quality and too short reads will first be discarded. As with other services, **FastQC** and **Fastp** will be used for this purpose.
- **Mapping against a reference**: filtered reads are then mapped against the hg19 human assembly via **BWA**. The resulting SAM file is then transformed into a BAM file, which is then sorted and indexed via **SAMTools**.
- **Variant calling**: variants present in the BAM file are detected by means of **SAMTools** and **VarScan**.
- **VCF annotation**: VCF files generated in the previous step are annotated by means of **BCFTools** and **KGGSEQ**.
- **Metrics calculation**: relevant metrics and a final HTML report are generated via **PICARD** and **MultiQC**, respectively.

## Bioinformatics procedure

First of all, take the service and click on _**Add resolution**_ in [iSkyLIMS](https://iskylims.isciii.es/) after logging in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym. In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:

```shell
cd /data/ucct/bi/services_and_colaborations/IIER/human_genetics
ll -tr
```

Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered.

**Have a look at the last service that contains the word `RBPANEL` in its name, and take note of the number linked to this term.**

For the new service, your service acronym will be this number + 1. For example, if the last folder after executing `ll -tr` is `SRVIIER527_20251007_RBPANEL17_ggomezm_S`, the service acronym for your new service will be _**RBPANEL18**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools micromamba environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder:

```shell
cd /data/ucct/bi/services_and_colaborations/IIER/human_genetics/
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
* Next, specify `lowfreq_panel`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVIIERXXX_YYYYMMDD_RBPANELXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:

* `lablog_lowfreqpanel`: an executable file that creates the `00-reads` folder which will contain all the reads for the analysis.
* `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

Let's execute the `lablog_lowfreqpanel` file:

```shell
bash lablog_lowfreqpanel
```

After executing this file, if everything is OK, we can now proceed with the new BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the analysis.

```shell
bu-isciii scratch SRVIIERXXX.X
```

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVIIERXXX_YYYYMMDD_RBPANELXXX_researcher_S/ANALYSIS/DATE_ANALYSIS01_RBPANEL
```

Once we're inside, we can start with the analysis.

Please remember to load all the necessary modules:

```
module load Nextflow singularity
```

### Quality Control (FastQC)

- Move to the first folder: `cd *ANALYSIS01_RBPANEL/01-fastQC/`.
- Read the `lablog` file you'll find and execute it `bash lablog`. This will create an auxiliar script called `_01_fastqc.sh` and a `logs/` folder.
- Now you are ready to execute `_01_fastqc.sh`.

### Pre-processing (fastp)

- You can go on with this step while the first one is still running. Start with `cd 02-preprocessing`.
- Read the `lablog` carefully to check that everything is correct. Then, execute it `bash lablog`.
- Similar to the previous step, you will see a new `logs/` folder and an auxiliar script called `_01_fastp.sh`.
- Run `_01_fastp.sh` to execute [Fastp](https://github.com/OpenGene/fastp?tab=readme-ov-file#fastp).

### Post-processing Quality Control

- Now, move to `03-preprocQC/`
- **This is exactly the same as step 1**, but we will do a quality assessment of the reads previously filtered with fastp.

### Mapping

Once raw reads have been filtered for quality and length, you can proceed with the next step of the analysis, which is mapping the reads against a reference, which in this case will be the h19 assembly of the human genome.
- Move to `04-mapping`
- Read the `lablog` file you'll find and execute it by `bash lablog`. This will create several auxiliar scripts and a `logs/` folder:
  - `_01_bwa_mem.sh`: this script will run `bwa mem` to map the filtered reads against the reference, generating a SAM file.
  - `_02_samtools_view.sh`: runs `samtools view` to transform the SAM file into a binary BAM file.
  - `_03_samtools_sort.sh`: runs `samtools sort` to sort the BAM file.
  - `_04_samtools_index.sh`: runs `samtools index` to index the sorted BAM file.

### Variant calling

Your filtered reads have been mapped to a reference genome, but now you need to perform the variant calling procedure. To do so, you'll have to move first to `05-samtools`.
- Read the `lablog` file you'll find and execute it by `bash lablog`. This will create an auxiliar script and a `logs/` folder:
  - `_01_samtools_mpileup.sh`: runs `samtools mpileup` to generate a PILEUP file that is necessary for the subsequent steps related to variant calling.

Once this script has finished, you can move to `06-VarScan`.
- Read the `lablog` file you'll find and execute it by `bash lablog`. This will create several auxiliar scripts and a `logs/` folder:
  - `_00_varscan_mpileup2cns.sh`: runs `varscan mpileup2cns` to make consensus calls (SNP/indel) from a mpileup file, with a minimum allele frequency equal to 5%. **DO NOT RUN THIS SCRIPT, BUT THE NEXT ONE INSTEAD**.
  - `_01_run_varscan_mpileup2cns.sh`: runs the previous script by means of srun.
  - `_02_gzip.sh`: compresses the obtained VCF for its subsequent annotation.

### Variant annotation

Once the VCF files have been generated for all samples, they need to be annotated via **KGGSEQ**. To do so, move to `07-annotation`.
- Read the `lablog` file you'll find and execute it by `bash lablog`. This will create several auxiliar scripts and a `logs/` folder. Launch them sequentially:
  - `_01_bcftools_query.sh`: runs `bcftools query` so that the VCF files have a desired format, generating for each sample a `.table` file.
  - `_02_kggseq.sh`: runs kggseq, using the hg19 assembly of the human genome, for the annotation, including low-frequency variants (up to 1%), generating a `_annot.txt` file as a result.
  - `_03_final_table.sh`: unzips the txt file obtained in the previous step, replaces the original header from the `.table` file with the one from the file `header` (generating another file called `_header.table`), and then uses the auxiliary script called `merge_parse.R` to merge the content of the `_header.table` and `_annot.txt` files, generating a final `_all_annotated.tab` file, which will contain all identified variants for each sample and their corresponding annotations.
  - `_04_filter_final_table.sh`: takes the `.tab` file generated in the previous step and filters variants affecting the **RB1** gene, generating another file called `_rb1_annotated.tab`.

### Metrics

Finally, once variants have been annotated, you can move to `99-stats`.
- Read the `lablog` file you'll find and execute it by `bash lablog`. This will create several auxiliar scripts and a `logs/` folder:
  - `_01_picardHsMetrics.sh`: given the BAM file, this script runs `picard CollectHsMetrics` to collect metrics regarding the capture of certain regions of interest within the reference genome, defined by an interval file in Picard interval_list format. To do so, the `targets-xgen.interval_list` file from REFERENCES is used. A `_hsMetrics.out` file is generated from this procedure.
  - `_02_hsMetrics_all.sh`: this script adds a header into a `hsMetrics_all.out` file, and then adds the metrics obtained for each sample into this file.
  - `_03_run_multiqc.sh`: this script runs MultiQC so that a final HTML report is generated regarding the service.

## `RESULTS` folder

Once the service is finished, go to `RESULTS` and execute the corresponding `lablog_lowfreq_panel_results`. After running this file, you should always check the following files and folders:

- `multiqc_report.html`: a MultiQC HTML report containing all the relevant information obtained from `picard` and control quality.
- `hsMetrics_all.out`: a file containing the mapping metrics for all samples.
- `*all_annotated.tab`: a file containing all the annotated variants identified for each sample
- `*rb1_annotated.tab`: a file containing all the annotated variants identified for each sample, in this case only in relation to the RB1 gene.

---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **`finish`** module of buisciii-tools. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

    $ bu-isciii finish SRVIIERXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (in **`lowfreq_panel`**, results from `02-preprocessing` and `05-samtools` are deleted). Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/IIER/human_genetics/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **`bioinfo-doc`** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ bu-isciii bioinfo-doc SRVIIERXXX.X

This module will be executed twice. The first time, select the **`service_info`** option, and the next time select the **`delivery`** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

>[!WARNING]
>When running the `delivery` mode of the `bioinfo_doc` module, you will be asked for **delivery notes** and **email notes**. **THESE ARE NOT THE SAME THING**. After running the `service_info` mode of this module, you'll see a folder for the service will have been created in `bioinfo_doc`. There, you can for example create two files: `delivery_notes.txt` and `email_notes.txt`. Edit these two files, and add the following information in each one of them:
>* `delivery_notes.txt`: `Results were delivered in the SFTP.` (literally)
>* `email_notes.txt`: everything you want the researcher to be aware of.

## Low-freq panel report template

When everything is properly copied into the SFTP folder, you can put the update in the `update_servicios` channel's _Team Standup Bot_, with the following template:

- _**SRVIIERXXXXXX - RBPANELXX**_:
  - **Servicio**: Low-frequency variants detection and annotation for whole genome or sequencing panel (e.g. retinoblastoma gene panel)
  - **Número de muestras**: XXX
  - **Estado**: XXX
  - **Cobertura media**: XXX - XXX (**you can check this in the `hsMetrics_all.out` file**).
  - **Media del porcentaje de lecturas a >10X**: XX% (**you can check this in the `hsMetrics_all.out` file**).
  - **Observaciones**: indicate any relevant information regarding the service, as well as any anomalies detected. In general, you'll have to indicate here the **number of variants** identified for the **RB1 gene** for each sample, as well as the **most important gene feature**. This information can be extracted from each **`_rb1_annotated.tab`** file from `RESULTS`.