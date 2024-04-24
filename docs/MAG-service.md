# How to perform a MAG Service

This is a brief tutorial on how to perform a MAG Service as a member of the ISCIII's Bioinformatics Unit!

Usually, MAG is part of other services like [IRMA](lint/to/flu/service), [viralrecon](https://github.com/BU-ISCIII/BU-ISCIII/wiki/SARS-CoV-2-service), or [PikaVirus](/link/to/pikavirus/), so when performing a MAG service, it's likely that the service folder, resolution, and acronym are already created as part of these other services. 

The typical acronyms for a service that includes MAG analysis are:
- GENOME** (for example: GENOMEFLU, GENOMERSV, GENOMEEV...)
- VIRAL-DISCOVERY
- SARSCOV2

To perform this manual, we will assume that the service only contains MAG and we will start from the beginning.

Now, considering you've already created a buisciii-tools conda environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder. As MAG is a metagenomic pipeline, it can be used in any of the ISCIII areas. We will assume this service was requested by a virology lab:

```
cd /data/bi/services_and_colaborations/CNM/virology/
```

Now, let's execute the first BU-ISCIII tool: `new-service`, where you'll need to specify the resolution ID associated to this service.

```
bu-isciii new-service SRVCNMXXX.X
```

Once `new-service` is executed, you'll be asked:

_Do you want to skip folder creation?:_ Unless it is not the first resolution associated with the service, answer NO, because the folder corresponding to the service has not yet been created in the services_and_collaborations folder.

Next, specify `mag_met`, since this is the service we're running. As I said before, as this will be part of other services, generally it will be copied from selecting `all` in the list of available services.

Once the `new-service` is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_ACRONYMXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iskyLIMS](https://iskylims.isciii.es/)

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP` (as explained [here](https://github.com/BU-ISCIII/BU-ISCIII/wiki/bioinformatics#33-services_and_collaborations)). We should check, before going any further, that the number of files contained within the RAW folder is equal to the number of samples specified in [iskyLIMS](https://iskylims.isciii.es/) x 2, in the case that they are paired-end reads.

If everything is OK, we can get into the ANALYSIS folder and we'll find the following items inside:
- `lablog_mag`: an executable file that creates the 00-reads folder, moves inside, creates symbolic links to the reads and renames the `ANALYSIS0X_MET` folder
- `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.
- `ANALYSIS0X_MAG`: Folder with the main MAG analysis files.

First of all, let's check in the `lablog_mag` if the renaming of the `ANALYSIS0X_MET` is correct. If the MAG analysis is the only analysis in the service, the folder will be `DATE_ANALYSIS01_MET`, else, you will have to sum as many numbers as other analysis you have prior to MAG. Usually it is `DATE_ANALYSIS02_MET`

Now we can execute this lablog:

```

bash lablog_mag
```

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `/scratch` folder.

> [!WARNING]
> If MAG is not the only analysis in your service, don't forget to run the other `lablogs` before the next steps.

```
bu-isciii scratch SRVCNMXXX.X
```

Once scratch is executed, you'll be asked:

- _Direction of the service:_ in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```
cd /data/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_ASSEMBLYXXX_researcher_S/ANALYSIS/DATE_ANALYSIS0X_MAG
```

Once we're inside, we can execute our next executable file: `lablog`, which will create symbolic links to our raw reads and the samples_id.txt file, and create the `mag.sbatch` and `_01_run_mag.sh` files.

```

bash lablog
```

Now, we should check we've loaded all the needed dependencies and perform the metagenomic analysis:

```
module load Nextflow singularity
bash _01_run_mag.sh
```

After this, the analysis will start, and we'll be able to check the status of the process with:

```
squeue -u youruser
```

Once checked everything has finished OK in the DATE_mag.log, it's time to execute the content of the `99-stats` folder. We need to create this folder because default MAG's MultiQC does not take Kraken2's output into the final report.

```
cd 99-stats
bash lablog
module load MultiQC
bash _01_run_multiqc.sh
```

Once this MultiQC process has finished, we can execute this other script to clean the folder:

```
bash _02_unlink.sh
```

Once all the process is finished, within the DATE_ANALYSIS0X_MAG folder, we'll have the following content:
- DATE_mag/
- 99-stats/multiqc_report.html: HTML report with the top5 species found in kraken2 among all samples.
