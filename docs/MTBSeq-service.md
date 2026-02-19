# How to perform the MTBSeq Service

Welcome to this (as) brief (as possible) tutorial on how to perform an **MTBSeq Service** as a member of the ISCIII's Bioinformatics Unit!

This service consists in an in-depth analysis of *Mycobacterium*-especies genomes, by means of the [**MTBSeq**](https://github.com/ngs-fzb/MTBseq_source) software, which is a pipeline for mapping, variant calling and detection of resistance-mediating variants from NGS data of *Mycobacterium* isolates.

First of all, take the service and click on _**Add resolution**_ in [**iSkyLIMS**](https://iskylims.isciii.es/) after logging in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym. In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/bacteriology
ll -tr
```

Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered. Have a look at the last service that contains the word `MTUBERCULOSIS` in its name, and take note of the number linked to this `MTUBERCULOSIS` term.

For the new service, your service acronym will be this number + 1. For example, if the last folder after executing `ll -tr` is `SRVCNM1247_20241106_MTUBERCULOSIS75_lherrera_S`, the service acronym for your new service will be _**MTUBERCULOSIS76**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools conda environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/bacteriology/
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

* `Do you want to skip folder creation?`: Unless it is not the first resolution associated with the service, answer **NO**, because the folder corresponding to the service has not yet been created in the `services_and_collaborations` folder. However, please consider that **this MTBSeq service is usually performed along with the assembly service**, so usually the service folder will have already been created.
* Next, specify `mtbseq`, since this is the service we're running (you'll already have indicated `assembly_annot` previously when running `new-service` for the first time).

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_MTUBERCULOSISXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:

* `lablog_mtbseq`: an executable file that creates the `00-reads` folder which will contain all the reads.
* `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

Let's execute the `lablog_mtbseq` file:

```shell
bash lablog_mtbseq
```

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the MTBSeq analysis.

```shell
buisciii scratch SRVCNMXXX.X
```

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_MTUBERCULOSISXXX_researcher_S/ANALYSIS/DATE_ANALYSIS02_MTBSEQ
```

> [!WARNING]
> Please note that the MTBSeq service is usually performed along with the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service); that's why this folder is called `ANALYSIS02`. During this assembly service, **we should have saved the trimmed sequences**, since **they will be needed** during the MTBSeq analysis.


Once we're there, and before executing anything else, we should load all the necessary dependencies:

```shell
module load singularity
```

Once we're inside this folder, we'll find the following elements:

* `lablog`: please execute this lablog file **first**, so that symbolic links for the `00-reads` folder and `samples_id.txt` are created.

* `01-preprocessing`: this subfolder contains a `lablog` file, which will ask us whether we saved the trimmed reads from the previous assembly procedure or not. If we say yes, symbolic links for these trimmed reads will be created inside this folder. If we reply no, an executable file called `_01_fastp.sh` will be created, which we will have to execute in order to obtain the trimmed sequences from the original reads, fetched from the `00-reads` folder.
  
* `02-kmerfinder`: this folder contains a `lablog` file that **should be executed only if, for some reason, the assembly procedure has NOT been performed previously**. **Otherwise, it is NOT necessary to execute this lablog**. If you execute this lablog, you'll get three .sh files:
  * `_01_kmerfinder.sh`: this script will run **Kmerfinder**, a program for bacterial species identification, for the trimmed sequences.
  * `_02_find_common_reference.sh`: this script extracts the most common reference for all the samples being analysed.
  * `_03_download_reference.sh`: this script downloads that reference within the REFERENCES folder of the service.

* `03-MTBSeq`: inside this folder, we'll find the following elements:
  * `lablog`: this file, once executed, will create the following scripts, that will have to be run sequentially:
  
    * `_00_prepareRaw.sh`: this script reads the `samples_id.txt` file, creates a subfolder for each sample and generates symbolic links for the trimmed reads of each sample. These symlinks will have the suffix *lib1* added in their names.
  
    * `_01_preparesamples.sh`: this script reads the `samples_id.txt` file and creates another file, inside the sample's corresponding subfolder, called `samples.txt`, which has two tab-separated columns, one indicating the sample name and the other one indicating the library (*lib1*).
  
    * `_02_mtbseq.sh`: this script launches the whole **MTBSeq** pipeline (by means of the `--TBfull` option) for each sample. This is performed using the `samples.txt` file as input.
  
    * `_03_gather_results.sh`: once MTBSeq has been executed, each sample folder will contain several subfolders (Amend, Bam, Called, Classification, Statistics, etc.), apart from a .log file. In order to have more accessible the information that we are interested in, this script can be run so that the following elements are created:
        * A subfolder called `classification_all`, in which we'll find a file called `strain_classification_all.tab`.
        * A subfolder called `resistances_all`, in which we'll find several .tab files, one for each sample.
        * A subfolder called `stats_all`, in which we'll find a file called `statistics_all.tab`.
  
  * `all_samples`: this folder contains a `lablog` file that:
    * Creates subfolders called `Amend`, `Bam`, `Called`, `Classification`, `GATK_Bam`, `Groups`, `Joint`, `Mpileup`, `Position_Tables` and `Statistics`.
  
    * Merges the content of the `samples.txt` files into one `samples.txt` file.
  
    * Creates symbolic links to the folders that were created initially.
  
    * Creates the following scripts, that should be run sequentially:
        * `_01_tb_join.sh`: this script runs the **TBjoin** module for all samples, creating a comparative SNP analysis. According to the [**MTBSeq manual**](https://github.com/ngs-fzb/MTBseq_source/blob/master/MANUAL.md), first a scaffold of all variant positions is built from the individual variant files. Second, for all positions with a variant detected for any of the input samples, the allele information is recalculated from the original mappings to produce a comprehensive table.
  
        * `_02_tb_amend.sh`: this script runs the **TBamend** module for all samples, for post-processing of joint variant tables. This step will produce a comprehensive variant table including calculated summary values for each position. In addition, the set of positions will be processed in consecutive filtering steps. In the first step, positions will be excluded if less than a minimum percentage of samples have unambiguous base calls at this position, with this threshold set by the `--unambig` OPTION (default: 95). In addition, all samples need to have either a SNP or wild type base at the position, and positions within repetitive regions of the reference genome or within a resistance associated genes are excluded. This filtering step results in output files carrying the “_amended__[unambig]_phylo” ending; a full table (ending in .tab), a FASTA file containing the aligned alleles of all samples for the given position (.fasta), and a corresponding FASTA file with the headers consisting solely of the respective sample ID (_plainIDs.fasta). The second filtering step removes positions that are located within a maximum distance to each other in the same sample, with this threshold set by the `--window` OPTION (default: 12). Output files from this filtering step are generated with the same naming scheme as for the first step and carry the selected window threshold as additional field. In addition, positions not passing the window criteria are reported in the output file carrying the tag "removed".

        * `_03_tb_groups.sh`: this script runs the **TBgroups** module for all samples, for inferring likely related isolates based on pairwise distance of distinct SNP positions. In an agglomerative process, samples are grouped together if they are within the threshold set by the `--distance` OPTION (default: 12). The output files consist of a text file listing the detected groups and ungrouped isolates, and the calculated pairwise distance matrix.

        * `_04_iqtreeall.sh`: this script runs **iqtree**, a phylogenetic tree creation programme, using the file ending in `amended_u95_phylo_w12.plainIDs.fasta` as input and employing the K3Pu+F+I DNA substitution model (check [**iqtree's documentation**](http://www.iqtree.org/doc/) for more info). A `.treefile` from this procedure will be obtained, corresponding to the created phylogenetic tree itself, which can be then loaded into a visualiser like [**iTOL**](https://itol.embl.de/).

    > [!WARNING]
    > This `all_samples` folder is of interest **ONLY** when the researcher asks us to perform a **comparative genomics analysis**, to wit, to perform **SNP analysis**. If the researcher does NOT mention this in their petition, we **DO NOT** have to run the lablog file from `all_samples`; we can simply **ignore** this folder.

Once all scripts, from the **Assembly** and the **MTBSeq** services, have been run, we can go to the `RESULTS` folder of the service, and we should have the scripts `lablog_assembly_results` and `lablog_mtbseq_results` there. The first lablog is already described in the [Assembly service](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) wiki page, while the second one creates, within the `DATE_entrega01` folder, a subfolder called `mtbseq`, inside which we'll find symbolic links to:
* The `resistances_all` folder.
* The `statistics_all.tab` file.
* The `strain_classification_all.tab` file.
* The `Amend` folder from `all_samples` (only if we have performed SNP analysis).
* The `Groups` folder from `all_samples` (only if we have performed SNP analysis).
* The .treefile obtained from iqtree (only if we have performed SNP analysis), saved as a .nwk file that can be loaded into a tree visualiser.

---

### What should I pay attention to before the delivery of the service?

Before we deliver the service, we should check the files that we have inside the `RESULTS` folder.
* `statistics_all.tab`: firstly, we should check that the info as a whole makes sense. Then, we should mainly check the column `%Mapped Reads`: if the percentage is considerably low, the sample might be contaminated or there might be other organisms. We should also check the columns:
  * `Genome GC`: GC content of the reference genome.

  * `(Any) Total Bases`: number of reads covering the reference genome. The closer to the reference genome size the better.
  
  * `(Any) Coverage mean`: mean coverage depth. The higher the better.
  
* `strain_classification_all.tab`: check that the info as a whole makes sense. Take into account that this .tab file contains information regarding lineage classification of the samples based on three different references (Homolka et al., 2012; Coll et al., 2014; Merker et al., 2015). Within this file, the majority lineage is reported for each sample. This entry also gives an indication of the data quality for the positions used to infer the phylogenetic classification. The column indicating the quality will contain the label _good_ if all phylogenetic positions contained in the set are covered at least 10fold and show a frequency of at least 75%, _bad_ if any phylogenetic position does not meet these, and _ugly_ if any phylogenetic position does not have a clear base call. <br><br>Considering this, check the columns:
  * `Homolka species`: check if the species assigned to each sample match the one indicated by Kmerfinder.
  
  * `Homolka lineage`.
  
  * `Quality`: check the lineage assignment quality.
  
  * `Coll lineage`.
  
  * `Coll quality`: check the lineage assignment quality.
  
  * `Beijing lineage`.
  
  * `Beijing quality`: check the lineage assignment quality.

* `resistances_all`: check that a .tab file, regarding resistance-associated variants, is stored for every sample within this folder.

* `Amend`: check that the symlink to this folder has been created correctly. Inside this folder, there should be a .tab file starting with *NONE* and ending with *amended.tab*. This file shows all the found variants and whether they are linked to resistance or not.

* `Groups`: check that the symlink to this folder has been created correctly.</br></br>
---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools.

    $ buisciii finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (in **MTBSeq**, these folders are `Bam`, `Mpileup` and `GATK_Bam`, along with the .fastq.gz files from `01-processing/fastp`). Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ buisciii bioinfo-doc SRVCNMXXX.X

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution. **Please indicate any incidences you might have found during the analysis in these notes**.

>[!WARNING]
>When running the `delivery` mode of the `bioinfo_doc` module, you will be asked for **delivery notes** and **email notes**. **THESE ARE NOT THE SAME THING**. After running the `service_info` mode of this module, you'll see a folder for the service will have been created in `bioinfo_doc`. There, you can for example create two files: `delivery_notes.txt` and `email_notes.txt`. Edit these two files, and add the following information in each one of them:
>* `delivery_notes.txt`: `Results were delivered in the SFTP.` (literally)
>* `email_notes.txt`: everything you want the researcher to be aware of.

Lastly, remember to remove all the files related to this service from `scratch_tmp`:

    $ buisciii scratch SRVCNMXXX.X
    $ remove_scratch

---

## MTBSeq report template

**_SRVCNMXXXX - SERVICE ALIAS_**:

* **Servicios**: Ensamblado de novo, MTBseq *(y análisis de SNPs)*.
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
  * **MTBseq**:
    * **Stats**: %mapped reads: XXXX.
    * **Classification**: Check whether the classification of the samples by MTBSeq matches the one provided by Kmerfinder.
* **Incidencias muestras individuales**: indicate any incidences that might be relevant, like samples that could not be assembled or samples for which the lineage could not be determined.

---

