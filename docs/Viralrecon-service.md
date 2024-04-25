# Viralrecon general service description:

Once the service has been [accepted in iSkyLIMS](https://github.com/BU-ISCIII/BU-ISCIII/wiki/How-to-manage-services-in-iSkyLims) and we have a Resolution ID (SRVCNMXXX.X), we can use the buisciii-tools to initialize the service itself.

Log in with your HPC user.

Load the buisciii-tools environment.

    $ conda activate buisciii-tools

Create the service and the needed folder structure. Select the **Viralrecon** template.

    $ bu-isciii new-service SRVCNMXXX.X
    > Viralrecon

> Note: If the resolution ID is not specified, it will be requested via the prompt.

If the service configuration is correct and the sequences are located in `/srv/fastq_repo`, move inside the newly created folder at `/data/bi/services_and_colaborations/CNM/virology/`. Check the `/RAW` folder to verify that symbolic links have been correctly created for all service samples.

Move to `/ANALYSIS`. Configure the `samples_ref.txt` file according to the service requirements (samples, reference genomes, hosts, etc.). Check the lablog and execute it.

    $ bash lablog_viralrecon

Este script consulta al usuario a través del prompt el tipo de análisis que va a llevarse a cabo (AMPLICONS o METAGENOMICS), establece los archivos de configuración de la carpeta `../DOC` conforme a esta decisión y crea una carpeta por cada hospedador que haya sido especificado en `samples_ref.txt` (normalmente solo 1).

This script prompts the user for the type of analysis to be performed (AMPLICONS or METAGENOMICS), sets up the configuration files in the `../DOC` folder and creates a folder for each host specified in samples_ref.txt (usually only 1).

> Note: In case the service has special requirements, additional configurations may be necessary.

Edit the folder name `YYMMDD_ANALYSIS_0X_MAG` based on the number of analyses to be performed. If only one host exists, it shall be set as `YYMMDD_ANALYSIS_02_MAG`.

Copy the contents of the service folders to scratch. To do this, run the **scratch** tool from buisciii-tools.

    $ bu-isciii scratch --direction service_to_scratch SRVCNMXXX.X

Once finished, move to the newly copied service folder in scratch (its mounted path in scratch_tmp) `/data/bi/scratch_tmp/bi/`. Access the `/ANALYSIS` folder and at this point, you will need to launch the pipeline once for each host successively. Access the folder of the first existing host (e.g., `YYYYMMDD_ANALYSIS01_METAGENOMIC_HUMAN`).

Check the lablog and execute it.

    $ bash lablog

This script generates different scripts that must be executed in an orderly manner according to their numbering. In first place, it generates a script for each reference, with the name `_01_run_<reference1>.sh.` If there is more than one reference, then there would be several files whose name begins with \_01_run. Execute these scripts first.

> Note: remember to load the necessary modules and environments specified in the lablog, for the correct execution of this script and the following ones.

    $ bash _01_run_<reference1>.sh

Running this script will spawn the pipeline in the SLURM process queue. You can monitor the pipeline progress by checking the newly generated log file.

    $ tail -f <reference>_date_viralrecon.log

Simultaneously, in another window, you can monitor the processes in the slurm queue for your user as follows:

    $ watch squeue -u <youruser>

Once the pipelines have been executed for all references, you can continue with the orderly execution of the following scripts (\_02, \_03, etc.). This will generate files collecting the results of the analyses performed.

> Note: It is not always necessary to execute all the scripts. \_05_create_stats_assembly.sh will only be necessary in cases where de novo assembly has been carried out.

Once finished, repeat the process if there are any other host.

---

On completion of the pipeline, it is strongly recommended to review different files to check that it all worked properly. Among others, the 3 essential ones are the following:

* `./<reference>_yyyymmdd_viralrecon_mapping/multiqc/multiqc_report.html`. Multiqc report provides a general overview of the data (variant calling metrics, sequencing quality, trimming info, coverage distribution, etc.).

* `./mapping_illumina_yyyymmdd.tab`. This file gathers relevant information about the mapping of the sequences with respect to the references.

* `./<reference>_yyyymmdd_viralrecon_mapping/variants/ivar/variants_long_table.csv`. This file contains information about the annotation of the identified variants for all samples with respect to each reference.

---

Meantime, you can access the `YYMMDD_ANALYSIS_0X_MAG` folder and execute the process following its [manual](https://github.com/BU-ISCIII/BU-ISCIII/wiki/MAG-service).

If the pipeline has **successfully finished**, move to the `../RESULTS` folder.

Check the lablog and execute it.

    $ bash viralrecon_results

Access the newly created folder and execute the scripts in order.

If everything is correct and the necessary files and links have been generated, you can proceed with the service completion. To do this, execute the finish module of buisciii-tools.

    $ bu-isciii finish SRVCNMXXX.X

This module will do several things. First, it cleans up the folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space. Then it copies all the service files back to its `/data/bi/services_and_colaborations/CNM/virology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the bioinfo-doc module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/bioinfo_doc/` folder.

    $ bu-isciii bioinfo-doc SRVCNMXXX.X

This module will be executed twice. First time select the service_info option, and the next time select the delivery option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

Lastly, remember to remove all the files related to this service from `scratch_tmp`:

    $ bu-isciii scratch SRVCNMXXX.X
    > remove_scratch

### Common errors while running the service

Make sure that `samples_ref` file does not contain any spaces as it's read as a tab-separated file. You can use `cat -A samples_ref.txt` to ensure this (`\t` is shown as `^I` in this case).

When there's a mix of full-numerical and strings in sample IDs (e.g. `87439.fastq.gz` and `SARS_01.fastq.gz`) the pipeline may crush in `MULTIQC` step. This is caused because there's a bug with MultiQC ([MultiQC issue](https://github.com/nf-core/viralrecon/issues/345)) that can be temporarily fixed by adding any non-numerical character to the sample IDs. Nevertheless, you can follow the instructions in [this tutorial](https://drive.google.com/drive/u/0/folders/1-GafpZR2HVlecNaAsXIslK3aecHplD4z) to properly correct this error.



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