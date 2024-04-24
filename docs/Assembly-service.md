# How to perform the Assembly Service
Welcome to this (as) brief (as possible) tutorial on how to perform an Assembly Service as a member of the ISCIII's Bioinformatics Unit!

First of all, take the service and click on _**Add resolution**_ in [iskyLIMS](https://iskylims.isciii.es/) after loggin in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym. In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:

```
cd /data/bi/services_and_colaborations/CNM/bacteriology
ll -tr
```
Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered. Have a look at the last service that contains the word `ASSEMBLY` in its name, and take note of the number linked to this `ASSEMBLY` term. 

For the new service, your service acronym will be this number + 1. For example, if the last folder after executing `ll -tr` is `SRVCNM1120_20240418_ASSEMBLY482_lherrera_S`, the service acronym for your new service will be _**ASSEMBLY483**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools conda environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user:
```
ssh -X -p 32122 youruser@portutatis.isciii.es
```

Once you're logged in, go into the `services_and_colaborations` folder:

```
cd /data/bi/services_and_colaborations/CNM/bacteriology/
ll -tr
```

Now, let's execute the first BU-ISCIII tool: `new-service`, where you'll need to specify the resolution ID associated to this service.

```
bu-isciii new-service SRVCNMXXX.X
```
Once `new-service` is executed, you'll be asked:
* `Do you want to skip folder creation?`: Unless it is not the first resolution associated with the service, answer **NO**, because the folder corresponding to the service has not yet been created in the `services_and_collaborations` folder.
* Next, specify `assembly_annot`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_ASSEMBLYXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iskyLIMS](https://iskylims.isciii.es/)

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iskyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:
* `lablog_assembly`: an executable file that checks whether our reads are paired-end or single-end and creates the 00-reads folder which will contain all the reads.
* `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

Let's execute the `lablog_assembly` file:

```
bash lablog_assembly
```

After executing this file, if everything is OK, we can now proceed with the new BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/bi`, since this `scratch_tmp` folder will be the one used for the assembly analysis.

```
bu-isciii scratch SRVCNMXXX.X
```

Once `scratch` is executed, you'll be asked:
* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```
cd /data/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_ASSEMBLYXXX_researcher_S/ANALYSIS/DATE_ANALYSIS01_ASSEMBLY
```

Once we're inside, we can execute our next executable file: `lablog`, which will create symbolic links to our raw reads and the `samples_id.txt` file, apart from asking us the following information:
* `Indicate the preferred assembly mode: short, long or hybrid`.
* `Do you want to save trimmed reads?`: Unless the researcher asks us to keep the trimmed reads, we can reply `NO`.
* `Is Gram - or +?`: Since the researcher tells us the organism, we can check online whether this bacterium is Gram - or +.

```
bash lablog
```

This information will be contained within the `samplesheet.csv` file within the `DATE_ANALYSIS01_ASSEMBLY` folder. Now, we should check we've loaded all the needed dependencies and perform the assembly analysis:

```
module load Nextflow/23.10.0 singularity
sbatch assembly.sbatch
```

After this, the analysis will start, and we'll be able to check the status of the process with:

```
squeue -u youruser
```

Once the process is finished, within the `DATE_ANALYSIS01_ASSEMBLY` folder, we'll have the following content:
* `01-processing`: all files associated with the quality control analysis performed by `fastp` and `fastqc`.
* `02-taxonomy_contamination`: a group of folders, with one folder containing the kmerfinder results for each sample.
* `03-assembly`: a folder containing the results from `quast` and `unicycler`.
* `05-annotation`: a folder contaning the results from annotation using PROKKA.
* `99-stats`: a folder containing a .csv file summarizing the results from kmerfinder and quast.
* `pipeline-info`: a folder containing reports about the execution of the assembly pipeline.
* `_01_nf_assembly.sh`: an executable file that performs the assembly analysis (`bash _01_nf_assembly.sh` does the same as `sbatch assembly.sbatch`).
* `DATE_assembly01.log`: a file reporting whether all the procedures of the pipeline have correctly been executed or not.
* `assembly.sbatch`
* `lablog`
* `samplesheet.csv`

After checking that the pipeline has been executed correctly and we have all the files we should have (mainly the kmerfinder and quast results), we can now go to the `RESULTS` folder, still inside `scratch_tmp`, and execute the following file, which creates symbolic links to our kmerfinder and quast reports, apart from our raw reads:

```
bash lablog_assembly_results
```

Once we've executed this, we'll have a new folder within the `RESULTS` folder named `DATE_entrega`, which will contain, at the same time, a folder named `assembly`. Inside `assembly`, we should have a symbolic link (called `assemblies`) linked to the `unicycler` folder containing all the raw reads, and other symbolic links associated with the kmerfinder `.csv` summary, all `quast` reports and the `multiqc` report, similarly to this:

```
assemblies              quast_GCF_000020105.1_ASM2010v1_report.html   summary_assembly_metrics_mqc.csv
kmerfinder_summary.csv  quast_GCF_000306985.1_ASM30698v1_report.html
multiqc_report.html     quast_global_report.html
```

If everything is correct and all the files have the expected content, we can proceed to copy the content of the `RESULTS` folder to the researcher's SFTP. To do this, we should now execute the next BU-ISCIII tool: `finish`, which will delete temporary files, copy the results from scratch back to the `services_and_colaborations` folder, rename those folders that should not be copied into the researcher's SFTP and copy those that are of interest to this SFTP:

```
bu-isciii finish SRVCNMXXX.X
```

After executing `finish`, you'll have to specify again that we are performing an assembly analysis (`assembly_annot`) and allow for this tools to rename (`RAW` and `TMP` will be renamed as `RAW_NC` and `TMP_NC`) and delete some folders (`work` will be deleted).

Once `finish` is done, the results will be now at the researcher's SFTP and we can go back to `/data/bi/services_and_colaborations/CNM/bacteriology/SRVCNMXXX_YYYYMMDD_ASSEMBLYXXX_researcher_S/RESULTS`. If all the reports have been copied correctly into the corresponding `services_and_colaborations` folder, we can now execute the next BU-ISCIII tool: `bioinfo-doc`, which will create a `.pdf` report with the information that will be delivered to the researcher.

To execute `bioinfo_doc`, we have to **go back to our WS user**, in which we should already have mounted the `bioinfo_doc` folder. If this is the case, we can do the following, **always after having checked the kmerfinder, quast and multiqc reports and looking for any remarkable aspects that the researcher should be informed about**:

```
bu-isciii bioinfo-doc SRVCNMXXX.X > service_info
```

Once you've specified the `service_info` option, you should execute the `bioinfo-doc` BU-ISCIII tool again indicating the `delivery` option this time. Please note that the program will ask you to create the markdown files associated to this specific delivery, apart from whether we want to add some notes. If there is something we want to inform the researcher about, we can create a `delivery_notes.txt` with this information, by executing the following commands, **before executing `bu-isciii bioinfo-doc SRVCNMXXX.X > delivery`**:

```
cd /data/bioinfo_doc/services/2024/SRVCNMXXX_YYYYMMDD_ASSEMBLYXXX_researcher_S #Go to the specific folder of the service.
nano delivery_notes.txt #Create and edit the .txt file with notes. Press Crtl + X to save your data and press Enter to exit.
```

Once this has been done, execute `bu-isciii bioinfo-doc SRVCNMXXX.X > delivery`, agree in the fact that you want to create markdown files, indicate the absolute path to the `delivery_notes.txt` file and agree to send an email with the results automatically to the researcher. `Bioinfo-doc` will have finished and a report with the results and your notes will have been sent to the researcher directly. The service is now delivered and finished.

Lastly, remember to remove all the files related to this service from `scratch_tmp`:

```
bu-isciii scratch SRVCNMXXX.X > remove_scratch
```