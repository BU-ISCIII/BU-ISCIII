# IRMA services

[IRMA (Iterative Refinement Meta-Assembler)](https://wonder.cdc.gov/amd/flu/irma/) was designed for the robust assembly, variant calling, and phasing of highly variable RNA viruses and is our go-to when analysing Influenza virus.
This software does not include quality trimming of the raw reads by itself. Therefore, before the execution of the pipeline, the samples are analysed with [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) for quality control and pre-processed with [Fastp](https://github.com/OpenGene/fastp?tab=readme-ov-file#fastp).

## Step-by-step guide

### 0. Create the service using [buisciii-tools](https://github.com/BU-ISCIII/buisciii-tools).

- Create the service from the local terminal using Iskylims Resolution ID (e.g. `bu-isciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X`). The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results). Type `N` to create the template folder and select the corresponding template folder.
- A folder will be created in `services_and_colaborations/CENTRE/SERVICE_TYPE` with the full name of the service, in the following format (YYYYMMDD = YearMonthDay):
`SRVCNMXXX_YYYYMMDD_GENOMEFLUXXX_researcheruser_S`
- Go to the recently created folder. Check that the number of reading files matches the number of samples that was specified in the service in iSkyLIMS (the number of files must be number of samples x 2 if they are paired, since there is a file of forward readings and one of reverse readings)
```
cd SRVCNMXXX_YYYYMMDD_WGSTRIOXXX_researcheruser_S/RAW
ls -l *.fastq.gz | wc -l
```
- If everything is alright, move to `ANALYSIS/` folder and execute `lablog` (This lablog might be named after the name of the template e.g. `lablog_irma`).
- You will end up having two folders inside `ANALYSIS/`, one named `ANALYSIS01_FLU_IRMA` and `ANALYSIS02_MAG`
- This first lablog will rename the ANALYSIS folders (`DATE_ANALYSIS01/02`) to the current date and create folder `00-reads` with symlinks to the fastq files in `RAW`.
- Finally, move the folder to the computing resource using `bu-isciii --log-file SRVCNMXXX.X.tool.log scratch --direction service_to_scratch SRVCNMXXX.X`. Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).
- From now on, all the analysis must be executed from the folder located in scratch.

### 1. Quality Control (FastQC)

- Move to the first folder `cd *ANALYSIS01_FLU_IRMA/01-preproQC/`.
- Read the lablog and execute it `bash lablog`. This will create an auxiliar script called `_01_rawfastqc.sh` and `logs/` folder.
- [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) is the software used in this step for quality control of the raw reads. You will need to load the module before executing with `module load`.
- Now you are ready to execute `_01_rawfastqc.sh`.

### 2. Pre-processing (fastp)

- You can go on with this step while the first one is still running. Start with `cd 02-preprocessing`.
- Read the lablog carefully to check that everything is correct. Then, execute it `bash lablog`.
- Similar to the previous step, you will see `logs/` folder and auxiliar script.
- Load the fastp module with `module load` and run `_01_fastp.sh` to execute [Fastp](https://github.com/OpenGene/fastp?tab=readme-ov-file#fastp)

### 3. Post-processing QC

- Move to `03-procQC/`
- This is exactly the same as step 1, but we will do a quality assessment of the reads previously filtered with fastp. 

### 4. IRMA

- In `04-irma/` you may find two files: `lablog` and `create_irma_stats.sh`. You should read them carefully before continuing with the analysis.
- Execute the lablog `bash lablog`, this will create several scripts `_01_irma.sh` `_02_create_stats.sh` and `_03_post_processing.sh`, along with `logs/` folder.
- Load the required modules with `module load` and execute the scripts sequentially.

### B-1. MAG

- [MAG](https://github.com/nf-core/mag) is used in this workflow to detect sources of contamination in the samples. You can read the [MAG_Documentation](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/MAG-service.md) for further description on the methodology of this analysis.

#### `RESULTS/` folder.

Once the service is finished, go to `RESULTS/` and execute the corresponding lablog. Depending on the type of Influenza Virus found int he samples you may find different files, but you should always check the following:

- `krona_results.html` includes the multiQC html report from [MAG](https://github.com/BU-ISCIII/buisciii-tools/blob/main/bu_isciii/assets/reports/md/mag.md)
- `all_samples_completo.txt` includes the fasta sequence for all the fragments in all the samples.
- Depending on the virus types found in your samples, you will have one folder for each fragment found (e.g. A_H1 for Influenza A H1_N1)
- In case of Influenza services (FLU), you will find a file named `flu_type_summary.txt`, which will include a summary of the different types of influenza found in your samples. This is the most relevant information to be included in your report.
