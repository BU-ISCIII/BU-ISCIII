# Viralrecon service description:

Once the service has been [accepted in iSkyLIMS](https://github.com/BU-ISCIII/BU-ISCIII/wiki/How-to-manage-services-in-iSkyLims) and we have a Resolution ID (SRVCNMXXX.X), we can use the buisciii-tools to initialize the service itself.

Log in with your HPC user.

Load the buisciii-tools environment (select the environment with the most recent version of buisciii-tools).

    $ micromamba env list | grep buisciii
    $ micromamba activate buisciii-tools_X.X.X

Create the service and the needed folder structure. Run twice the "new-service" module, selecting **Viralrecon** and **Taxprofiler** templates separately, in order to add both to the folder structure.

    $ bu-isciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X
    > Viralrecon (first time)
    > Taxprofiler (second time)

> Note: If the resolution ID is not specified, it will be requested via the prompt. The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results).

If the service configuration is correct and the sequences are located in `/srv/fastq_repo`, copy de log inside the newly created folder at `/data/ucct/bi/services_and_colaborations/CNM/virology/` and move in. Check the `/RAW` folder to verify that symbolic links have been correctly created for all service samples.

Move to `/ANALYSIS`. Configure the `samples_ref.txt` file according to the service requirements (samples, reference genomes, hosts, etc.). Check the `lablog_viralrecon` and execute it.

    $ bash lablog_viralrecon

This script prompts the user for the type of analysis to be performed (AMPLICONS or METAGENOMICS), sets up the configuration files in the `../DOC` folder and creates a folder for each host specified in samples_ref.txt (usually only 1).

> Note: In case the service has special requirements, additional configurations may be necessary.

Check the `lablog_taxprofiler` Edit the folder name `YYMMDD_ANALYSIS_01_TAXPROFILER` based on the number of analyses to be performed. If only one host exists, it shall be set as `YYMMDD_ANALYSIS_02_TAXPROFILER`.

Copy the contents of the service folders to scratch. To do this, run the **scratch** tool from buisciii-tools.

    $ bu-isciii --log-file SRVCNMXXX.X.tool.log scratch --direction service_to_scratch SRVCNMXXX.X

Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).

Once finished, move to the newly copied service folder in scratch (its mounted path in scratch_tmp) `/data/ucct/bi/scratch_tmp/bi/`. Access the `ANALYSIS` folder and at this point, you will need to launch the pipeline once for each host successively. Access the folder of the first existing host (e.g., `YYYYMMDD_ANALYSIS01_METAGENOMIC_HUMAN`).

Check the lablog and execute it.

    $ bash lablog

This script generates different scripts that must be executed in an orderly manner according to their numbering. In first place, it generates a script for each reference, with the name `_01_run_<reference1>.sh.` If there is more than one reference, then there would be several files whose name begins with \_01_run. Execute these scripts first.

> Note: remember to load the necessary modules and environments specified in the lablog, for the correct execution of this script and the following ones.

    $ bash _01_run_<reference1>.sh

---

**NOTE FOR AMPLICONS**

In the case of analysis of sequenced samples using amplicons, the primer_bed with the coordinates of the primers must be provided. If a de novo assembly is being carried out, it is also necessary to provide a fasta file with the sequences of the primers. These 2 files can be placed in the `../../REFERENCES` folder. **Before running the `_01_run_<reference1>.sh` script**, the following lines must be added to the end of the corresponding `<reference>_viralrecon.sbatch` file:

    --primer_bed ../../REFERENCES/Primers_scheme_rsv.bed \
    --primer_fasta ../../REFERENCES/RSV_primers.fasta

Furthermore, in the case of RSV analysis, the type of virus (A or B) must be specified by adding `rsv_a` or `rsv_b` in the next line:

    --nextclade_dataset_name 'rsv_x'

* Note: If you only have the sequence of the primers, you can create your own scheme.bed file using blast. Check [How to create scheme.bed](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Viralrecon-service#how-to-create-schemebed) for more information.

---

Running this script will spawn the pipeline in the SLURM process queue. You can monitor the pipeline progress by checking the newly generated log file.

    $ tail -f <reference>_date_viralrecon.log

Simultaneously, in another window, you can monitor the processes in the slurm queue for your user as follows:

    $ watch squeue -u <youruser>

Once the pipelines have been executed for all references, you can continue with the orderly execution of the following scripts (\_02, \_03, etc.). This will generate files collecting the results of the analyses performed.

> Note: It is not always necessary to execute all the scripts. \_05_create_stats_assembly.sh will only be necessary in cases where de novo assembly has been carried out.

Once finished, repeat the process if there are any other host.

> [!WARNING]
> If there is more than 1 host, please remember to use the appropriate kraken database. Full info on which organisms are associated with each kraken database can be found in **`/data/ucct/bi/references/kraken/README`**.

---

On completion of the pipeline, it is strongly recommended to review different files to check that it all worked properly. Among others, the 3 essential ones are the following:

* `./<reference>_yyyymmdd_viralrecon_mapping/multiqc/multiqc_report.html`. Multiqc report provides a general overview of the data (variant calling metrics, sequencing quality, trimming info, coverage distribution, etc.).

* `./mapping_illumina_yyyymmdd.tab`. This file gathers relevant information about the mapping of the sequences with respect to the references.

* `./<reference>_yyyymmdd_viralrecon_mapping/variants/ivar/variants_long_table.csv`. This file contains information about the annotation of the identified variants for all samples with respect to each reference.

---

**In the meantime, you can access the `YYMMDD_ANALYSIS_0X_TAXPROFILER` folder and execute the process following its [**manual**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Taxprofiler-service).**

If the pipeline has **successfully finished**, move to the `../RESULTS` folder.

Check the lablog and execute it.

    $ bash lablog_viralrecon_results

Access the newly created folder and execute the scripts in order.

> [!WARNING]
> If the researcher asked you to provide them with the **reads without host**, create a folder within `/RESULTS/*_entrega01/`, and copy these reads inside this folder, so that the researcher can find them easily.

If everything is correct and the necessary files and links have been generated, you can proceed with the service completion. To do this, execute the finish module of buisciii-tools.

    $ bu-isciii --log-file SRVCNMXXX.X.finish.log finish SRVCNMXXX.X

This module will do several things. First, it cleans up the folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space. Then it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/virology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the bioinfo-doc module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ bu-isciii --log-file SRVCNMXXX.X.tool.log bioinfo-doc SRVCNMXXX.X

Remember to save the logs with the corresponding name (i.e. `SRVCNMXXX.X.service_info.log` or `SRVCNMXXX.X.delivery.log`).

This module will be executed twice. First time select the service_info option, and the next time select the delivery option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

Lastly, remember to remove all the files related to this service from `scratch_tmp`:

    $ bu-isciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
    > remove_scratch

---

### How to create scheme.bed

`scheme.bed` is a file that contains coordinates of where your primers are located inside a reference in a table format. The columns are:

| Column Index  | Name | Description  
| ------------- | ------------- | ------------- |
| 1  | chrom  | The reference genome.  |
| 2  | chromStart  | Start position of the sequence  |
| 3  | chromEnd  | Ending position of the sequence  |
| 4  | name  | Primer name (make sure its the same as `primers.fasta`)  |
| 5  | primerPool  | Primer pool (irrelevant most of the times)  |
| 6  | strand  | Orientation of the strand |

If the reference selected for the service has changed, it is quite likely that your primers now land on different positions of the genome. Therefore, you will need to create a new bed file with the new coordinates.

To do so, you can use blast in order to align your sequences to your reference:
`blastn -num_threads 10 -evalue 1 -task 'blastn-short' -subject /data/ucct/bi/references/virus/RSV/your_reference.fasta -query primers.fasta -out blast.txt -outfmt '6 stitle std slen qlen qcovs' -num_alignments 1`

This will generate a series of alignments with your primers sequences in `blast.txt`. Lets add a header so it's easier to understand:

`echo 'stitle    qaccver saccver pident  length  mismatch        gapopen qstart  qend    sstart  send    evalue  bitscoreslen    qlen    qcovs' | cat - blast.txt > blast_mod.txt`

**Note:** You need only one row per primer in your scheme.bed but you will most likely find some queries that don't match exactly with the reference. In these cases you should try to select the ones with the `length` as close to the `qlen` as possible but keeping the `pident` as high as possible too (try to find a balance between the two metrics)

Once you have selected the corresponding lines, you can execute an auxiliar script called `blast_parser.py` (you may find it in `/data/ucct/bi/references/auxiliar_scripts/`) to do the rest of the work:
```python3 blast_parser.py blast_mod.txt scheme.bed```

---

### Common errors while running the service
* Make sure that `samples_ref` file does not contain any spaces as it's read as a tab-separated file. You can use `cat -A samples_ref.txt` to ensure this (`\t` is shown as `^I` in this case).
  
* When there's a mix of full-numerical and strings in sample IDs (e.g. `87439.fastq.gz` and `SARS_01.fastq.gz`) the pipeline may crush in `MULTIQC` step. This is caused because there's a bug with MultiQC ([MultiQC issue](https://github.com/nf-core/viralrecon/issues/345)) that can be temporarily fixed by adding any non-numerical character to the sample IDs. Nevertheless, you can follow the instructions in [this tutorial](https://drive.google.com/drive/u/0/folders/1-GafpZR2HVlecNaAsXIslK3aecHplD4z) to properly correct this error.

* When running `bash _01_run_<reference1>.sh` and checking the `.log` file, you might see the following message related with Bowtie2, where `XXX` is the number of mapped reads against the reference for each sample:
    > -[nf-core/viralrecon] X samples skipped since they failed Bowtie2 1000 mapped read threshold: <br> XXX: SAMPLE1 <br> XXX: SAMPLE2 <br> ... </br>
    
    If this happens, you'll have to determine if it is still worth it to analyse these samples (**we want to find out if the number of mapped reads for each sample (XXX above) is greater or equal to the theoretical number of reads needed for a 10x coverage of the reference genome**). To do so, do the following calculation:
    > A) **PAIRED-END READS**: number of reads = (genome size * 10) / (read length x 2) <br> B) **SINGLE-END READS**: number of reads = (genome size * 10) / (read length) </br>

    If the obtained number is greater than the one associated with each sample (XXX), this sample can be **omitted** in the analysis. Otherwise, it might be worth it to decrease `--min_mapped_reads` (default value: **1000**) when running viralrecon so that this sample is not skipped by Bowtie2.

* If the service contains samples from **multiple runs**, the corresponding  `./mapping_illumina_yyyymmdd.xlsx` will contain, in its first column, only one value: the name of the first run that was found by the script, which is used in this file for every sample even though that's not correct. Make sure to change this manually in this file so that every sample is associated with its correct run.

* If your service contains samples with single-end reads, the `%unmapped_reads` column in `./mapping_illumina_yyyymmdd.xlsx` will have **negative** values, which clearly makes no sense. If this happens, you'll have to do the following calculations on the following columns of this file, so that data is correct:
    * Column `readshostR1` must be equal to `readshost` for these samples.
    * `%readshost` = (`readshost` * 100) / `totalreads`
    * `unmappedreads` = `totalreads` - (`readshost` + `readsvirus`)
    * `%unmappedreads` = (`unmappedreads` * 100) / `totalreads`

* Make sure no **Ns** are found in the `variants_long_table.csv` file.

---

### What if the researcher wants me to send them the no host reads?

If the researcher, when requesting the service, asks you to provide them with the **.fastq.gz files with no host**, you'll have to add this on `/DOC/viralrecon.config`:

```
withName: 'KRAKEN2_KRAKEN2' {
            publishDir = [
                pattern: "*.{unclassified.fastq.gz,unclassified_1.fastq.gz,unclassified_2.fastq.gz,txt}"
            ]
        }
```
The no host reads will be inside **`/*_mapping/kraken2/`** for each host and each reference.

---

## Viralrecon report template

**_Viralrecon_**

**_SRVCNMXXX - SERVICE_ALIAS_**

[Service description and references used]

- **Número de muestras**: 10
- **Estado**: Finalizado y copiado al sftp
- **Instrumento y longitud**: Nextseq (2x150)
- **Cantidad de lecturas**: 0,5M - 2.4M
- **Calidad general**: Buena
- **Incidencia muestras individuales**:
    - 2 muestras no consiguen mapear (CONTROLNEGATIVO y POSUL54). Tienen muy pocas lecturas, baja calidad y elevado porcentaje de secuencias sobrerrepresentadas.
  

---