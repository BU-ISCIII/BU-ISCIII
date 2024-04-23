# ExomeEB-ExomeTrio-WGSTrio

These services are grouped together because they share most of the steps needed for the analysis. Nonetheless, they are aimed to different goals:
- ExomeEB stands for Exome in a single patient of [Epidermolysis bullosa](https://www.mayoclinic.org/diseases-conditions/epidermolysis-bullosa/symptoms-causes/syc-20361062).
- ExomeTrio is an Exome analysis from 3 related patients (usually Father, Mother and Children).
- WGSTrio is a Whole Genome Analysis of 3 (or more) related patients (usually Father, Mother and Children).

All these services use [Sarek](https://nf-co.re/sarek/3.4.0), a pipeline for Mapping, Variant Calling and Annotation of both Whole Genome and Targeted sequencing data specially designed for Human and Mouse (although it can work with any reference genome).

## Step-by-step guide.

### 1. Create the service using [buisciii-tools](https://github.com/BU-ISCIII/buisciii-tools)

- Create the service from the local terminal using Iskylims Resolution ID (e.g. `bu-isciii new-service SRVIIERXXX.X`). Type `N` to create the template folder and select the corresponding template folder (either exometrio, exomeeb or wgstrio).
- A folder will be created in `services_and_colaborations/CENTRE/SERVICE_TYPE` with the full name of the service, of the form (YYYYMMDD = YearMonthDay):
`SRVIIERXXX_YYYYMMDD_WGSTRIOXXX_researcheruser_S`
- Go to the recently created folder. Check that the number of reading files matches the number of samples that was specified in the service in iSkyLIMS (the number of files must be number of samples x 2 if they are paired, since there is a file of forward readings and one of reverse readings)
```
cd SRVIIERXXX_YYYYMMDD_WGSTRIOXXX_researcheruser_S/RAW
ls -l *.fastq.gz | wc -l
```
- `REFERENCES` folder will include BED files with coordinates for either regions, exons and genes depending on the template
- If everything is alright, move to `ANALYSIS` folder and execute `lablog` (This lablog might be named after the name of the template e.g. `lablog_exometrio`).
- This first lablog will rename the main ANALYSIS folder (`DATE_ANALYSIS01`) to the current date and create folder `00-reads` with symlinks to the fastq files in `RAW`.
- Finally, move the folder to the computing resource using `bu-isciii scratch --direction service_to_scratch SRVIIERXXX.X`.
- From now on, all the analysis must be executed from the folder located in scratch.

### 2. Analysis starts: Sarek

- Execute lablog inside `DATE_ANALYSIS01_SERVICE`. This lablog will create the following files:
  - `samplesheet.csv` containing the sample names necessary for sarek
  - `sarek.sbatch`, the script with default configuration to run sarek, which is launched with `_01_run_sarek.sh`
  - `../../DOC/family.ped` a file containing the pedigree of the samples in [PED format](https://gatk.broadinstitute.org/hc/en-us/articles/360035531972-PED-Pedigree-format).
     - You should check the `family.ped` file when there are more than 3 samples in the case of trios, as it might need some editing.

- As indicated in the lablog. Before running `_01_run_sarek.sh` you need to load the required modules: `module load Nextflow singularity`
- Then, you can run `bash _01_run_sarek` 
- Once sarek is finished, you can run `_02_clean.sh`, an auxiliar script that removes folders generated during the workflow which are not used in further steps of the analysis.

### 3. Post-processing of results from sarek.

- Enter the post-processing folder and execute the lablog: `cd 02-postprocessing; bash lablog`
- Load the corresponding modules that can be found in the lablog: `module load GATK/X.X.X...`
- Sequentially execute each of the scripts previously generated to filter the results from Sarek's variant calling (vcf):
  - `_01_separate_snps_indels.sh` separates SNPs and indels into two different vcf files: `snps.vcf.gz` & `indels.vcf.gz`
  - `_02_filter.sh` apply several filters to process the vcf files.
  - `_03_merge_vcfs.sh` merges the two previous vcf files and `_04_gzip.sh` compresses the merged vcf.
  - If everything went well, you should find a file named `variants_fil.vcf.gz`, which contains the merged filtered variants.
