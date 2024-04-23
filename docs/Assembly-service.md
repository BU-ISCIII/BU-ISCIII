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

To execute `bioinfo_doc`, we have to go back to our WS user, in which we should already have mounted the `bioinfo_doc` folder. If this is the case, we can do the following, **always after having checked the kmerfinder, quast and multiqc reports and looking for any remarkable aspects that the researcher should be informed about**:

```
bu-isciii bioinfo-doc SRVCNMXXX.X > service_info
bu-isciii bioinfo-doc SRVCNMXXX.X > delivery
```

After specifying the `delivery` option, the program will ask us to create the markdown files associated to this specific delivery, and will ask us if we want to add some notes. If there is something we want to inform the researcher about, 






--- 

Create the directory structure
```
mkdir ANALYSIS DOC RAW REFERENCES RESULTS TMP
```
Move to the RAW directory
```
cd RAW
```
Inside the Service Info Display page, check the name of the sequencing runs that are involved.

Create a symbolic link to the reads that will be used. 
```
ln -s /srv/fastq_repo/$RUNS .
```
Optionally, check how many reads there are, for future references and error tracking. Take into account that the samples might be either *paired-end* or *single-end*.
```
ls | wc -l
```

Move to the ANALYSIS directory
```
cd ../ANALYSIS
```
Record the assembly template path, we will copy all the lablogs from there (of course, this will only work inside the BU-ISCIII file system).

```
ASSEMBLY_TEMPLATE=/data/bi/pipelines/TEMPLATES/ASSEMBLY_TEMPLATE
```

Enter the ANALYSIS directory inside the service, and copy the corresponding lablog from the template. 

```
cp $ASSEMBLY_TEMPLATE/ANALYSIS/lablog .
```

<span style="color:red;"> Check the lablog </span> just in case (do always check the lablog to avoid disasters), then execute it.
```
bash lablog
```








This first lablog will create the following elements:
* <span style="color:#191970 ;">00-reads</span>: directory where the trimmed reads will be stored. 
* <span style="color: #191970;">$(date '+%Y%m%d’)_ANALYSIS01_ASSEMBLY</span>: directory where the analysis will be performed.

Enter this analysis directory.
```
cd *_ANALYSIS01_ASSEMBLY
```
Copy the lablog of this directory from the template.
```
cp $ASSEMBLY_TEMPLATE/ANALYSIS/ANALYSIS01_ASSEMBLY/lablog .
```
Check the lablog and execute.
```
bash lablog
```
The lablog will have created several elements: 
* <span style="color: #279406;">_01_nf_assembly.sh</span>: script to perform the assembly pipeline.
* <span style="color: #191970;">01-preprocessing</span>: directory where preprocessing with Trimmomatic will be performed.
* <span style="color: #191970;">02-kmerfinder</span>: directory where kmerfinder will be performed.
* <span style="color: #191970;">99-stats</span>: directory where kmerfinder results will be stored.
* <span style="color: #191970;">00-reads</span>: symbolic link to the 00-reads directory.
* <span style="color:#1414E0 ;">samples_id</span>: symbolic link to the sample list.
* <span style="color: #1414E0;">bacterial_qc</span>: symbolic link to a directory containing necessary scripts for analysis.

Move to the first step, 01-preprocessing:
```
cd 01-preprocessing
```
Copy the lablog from the 01-preprocessing of a previous service.
```
cp $ASSEMBLY_TEMPLATE/ANALYSIS/ANALYSIS01_ASSEMBLY/01-preprocessing*/lablog .
```
Check the lablog just in case, and execute it.
```
bash lablog
```
 it will create two elements:
 * <span style="color:#279406 ;">_01_preprocess.sh</span>: script to trim the reads with Trimmomatic in the HPC.
 * <span style="color:#279406 ;">_02_pgzip.sh</span>: script to compress the trimmed reads to fastq.gz format in the HPC.

Execute the first script:
```
bash _01_preprocess.sh
```
This will use the HPC to perform the trimming of the samples. If you wish to monitor the state of the submitted Jobs, please use `qstat` (or `watch qstat`, you can exit watch with `ctrl+c`). Once the trimming is finished, check the logs generated by the HPC in this directory. In this case, this command will allow direct confirmation of a successful trimming
```
grep -c "Completed successfully" TRIMMOMATIC.*.o*
```
A single coincidence per log is expected. Please bear in mind that only the files with `.o` extension will contain this message, whereas `.po` files will not. Once ensured, make a directory for the logs, and place all of them in it
```
mkdir logs
mv TRIMMOMATIC* logs
```
Execute the second script to compress the trimmed files
```
bash _02_pgzip.sh
```
Once finished, move the PGZIP to the logs (these are empty, or at least they should be)
```
mv PGZIP* logs
```
Go to the next part, 02-kmerfinder
```
cd ../02-kmerfinder
```
Again, copy the lablog from the 02-kmerfinder of the template

```
cp $ASSEMBLY_TEMPLATE/ANALYSIS/ANALYSIS01_ASSEMBLY/02-kmerfinder/lablog .
```

Check the lablog, and execute it
```
bash lablog
```

It will create three scripts:
* <span style="color: #279406;">_00_kmerfinder.sh</span>: will run kmerfinder in the HPC to find the correct reference for the service.
* <span style="color: #279406;">_02_find_common_reference.sh</span>: will *only* display the kmerfinder top-hits.
* <span style="color: #279406;">_03_download_reference.sh</span>: will download the most common top-hit and place it inside the REFERENCES directory.

Load the <span style="color:#1D315F;font-style: italic;">Singularity</span> module so kmerfinder can be executed
```
module load singularity/singularity-2.6.0
```
Execute the first script
```
bash _00_kmerfinder.sh
```
Create the directory for logs:
```
mkdir logs
```
Once the kmerfinder process is over, move the KMERFINDER logs to logs
```
mv KMERFINDER* logs
```
Execute the second script. This will display the top-hits for kmerfinder. Make sure to check if the top-hits make sense, otherwise check if everything has gone alright, or even contact the service solicitant.
```
bash _02_find_common_reference.sh
```
Execute the third script. This will download the most common top-hit assembly (the first one if tied), in faa, fna and gff format, and place all of them inside the references directory.

```
bash _03_download_reference.sh
```

Exit the directory.
```
cd ..
```
As we will run the nf assembly pipeline, we will activate the appropriate <span style="color: #40AB29; font-style: italic">conda</span> environment for it.
```
conda activate nextflow
```
Now, we will change the reference used in the `_01_nf_assembly.sh` script to the reference we just downloaded (dont include the `_genomic` part).
```
sed -i “s/@@@@@/your_reference/g” _01_nf_assembly.sh
```
Once changed, execute the script in the background (so it cant be interrupted if connection to the HPC is lost).
```
nohup bash _01_nf_assembly.sh &> $(date '+%Y%m%d')_assembly01.log &
```
While the previous script is running, go to 99-stats to generate the statistics of the kmerfinder step.
```
cd 99-stats
```
Copy the lablog from the 99-stats folder of the service template.
```
cp $ASSEMBLY_TEMPLATE/ANALYSIS/ANALYSIS01_ASSEMBLY/99-stats/lablog . 
```

Check the lablog and execute it (this one will do all the work).
```
bash lablog
```

Once all of these steps have ended, the service will be finished. However, there are some extra steps to take into account before delivery.

## Preparing the delivery

Before delivering the service, it is necessary to **remove not-so-useful and redundant data** (such as the trimmed reads). Go to the main directory of the service.
```
cd ../../..
```

Change the name of the `RAW` and `TMP` directories, adding a _NC (no copy) to their names
```
mv RAW RAW_NC
mv TMP TMP_NC
```
Then, go to `ANALYSIS/*_ANALYSIS01_ASSEMBLY`

```
cd ANALYSIS/*_ANALYSIS01_ASSEMBLY
```
The only files needed inside of `01-preprocessing` are the logs, and the lablog, so delete everything there except for those.
```
cd 01-preprocessing 
rm -rf -v !("lablog"|"logs")
``` 

Go back and change the name of `01-preprocessing`, adding a _DEL (delete)
```
cd ..
mv 01-preprocessing 01-preprocessing_DEL
```
The work directory generated by nextflow takes a lot of space, and its not necessary, so deleting it will safe us that precious disk capacity.
```
rm -rf work
```
Inside the `03-assembly` directory, there are some trimmed reads.
```
cd 03-assembly
rm -rf trimming/trimmed/*
```
Changing that `trimmed` directory to `trimmed_DEL` will end this part of the guide.
```
mv trimming/trimmed trimming/trimmed_DEL
``` 

With all of this, the service is ready to be delivered. However, the result revision is still pending.

## In short
Here, we gather all the above steps without an explanation, so the service can be launched in a blast. However, take into account that you should always check the lablogs.

ASSEMBLY_TEMPLATE="/data/bi/pipelines/TEMPLATES/NEW_ASSEMBLY_TEMPLATE"
mkdir $SERVICE_FOLDER_NAME && cd $_

mkdir ANALYSIS DOC RAW REFERENCES RESULTS TMP
cd RAW
*fill raw directory*

cp $ASSEMBLY_TEMPLATE/DOC/hpc_slurm_assembly.config DOC
cd ANALYSIS
cp $TEMPLATE/ANALYSIS/lablog .
bash lablog
ls
ls *ANALYSIS01_ASSEMBLY
bash _01_copy_folder.sh

cd /data/bi/scratch_tmp/bi/$SERVICE_FOLDER_NAME
cd ANALYSIS/*ANALYSIS01*
cp $ASSEMBLY_TEMPLATE/ANALYSIS/ANALYSIS01_ASSEMBLY/lablog .
bash lablog [ + / - ]
module load Nextflow singularity
bash _01_nf_assembly.sh



## Revising the results

Once all processes have ended (remember to always check this with `qstat`), its time to go over the results.

First of all, check the **quality of the reads**. This can be checked in the `03-assembly` directory. FastQC and MultiQC reports can be found there.

Secondly, check the **quality of the assembly**. A Quast report can be found in `03-assembly`.

Last, but not least, check the kmerfinder csv generated in `99-stats` to see if there are hints of **contamination**.

Make sure to take note of any anomalies. Sometimes something might be off, or might not make sense. Try to find a reason for this (wrong parameters in a process, not-so-good quality of the reads). Maybe, repeating the process with some changes is necessary, take note of that as well.



## Troubleshooting

Sometimes, incidences ocurr. Here, we register all troubleshooting to try and help solve it fast and easy.

**Wrong reference** (21-9-2021): sometimes, investigators provide their samples stating that they belong to a certain organism. The kmerfinder result might prove this right, or not. If the results agree with the investigator, the analysis can continue. Hoever, if it does not, please note it down before proceeding.   
	
	
**User namespace** (22-9-2021): 
Found in: kmerfinder process

Description: kmerfinder process stopped early, and logs showed the following message:

`ERROR : Failed to create user namespace: user namespace not supported by your system`

Solution: Restarting the ssh connection worked

**NEWUSER namespace runtime** (15-2-2022)
Found in: kmerfinder process

Description: kmerfinder process stopped early, logs showed the following message:

`Failed invoking the NEWUSER namespace runtime: Invalid argument`
`ABORT  : Retval = 255`

