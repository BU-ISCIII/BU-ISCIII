# ExomeEB-ExomeTrio-WGSTrio (sarek)

These services are grouped together because they share most of the steps needed for the analysis. Nonetheless, they are aimed to different goals:

- **ExomeEB** stands for Exome in a single patient of [Epidermolysis bullosa](https://www.mayoclinic.org/diseases-conditions/epidermolysis-bullosa/symptoms-causes/syc-20361062).
- **ExomeTrio** is an **Exome** analysis from 3 related patients (usually Father, Mother and Children), taking **inheritance** into account.
- **WGSTrio** is a **Whole Genome Analysis** of 3 (or more) related patients (usually Father, Mother and Children), taking **inheritance** into account.

All these services use [Sarek](https://nf-co.re/sarek/3.4.0), a pipeline for Mapping, Variant Calling and Annotation of either Whole Genome or Targeted sequencing data specially designed for Human and Mouse (although it can work with any reference genome).

## Step-by-step guide

### 1. Create the service using [buisciii-tools](https://github.com/BU-ISCIII/buisciii-tools)

- Create the service from the local terminal using Iskylims Resolution ID (e.g. `bu-isciii new-service SRVIIERXXX.X`). Type `N` to create the template folder and select the corresponding template folder (either exometrio, exomeeb or wgstrio).
- A folder will be created in `services_and_colaborations/CENTRE/SERVICE_TYPE` with the full name of the service, in the following format (YYYYMMDD = YearMonthDay):
`SRVIIERXXX_YYYYMMDD_WGSTRIOXXX_researcheruser_S`
- Go to the recently created folder. Check that the number of reading files matches the number of samples that was specified in the service in iSkyLIMS (the number of files must be number of samples x 2 if they are paired, since there is a file of forward readings and one of reverse readings)

```
cd SRVIIERXXX_YYYYMMDD_WGSTRIOXXX_researcheruser_S/RAW
ls -l *.fastq.gz | wc -l
```

- For exome services, `REFERENCES` folder should include BED files with coordinates for the targeted regions, exons and/or genes during sequencing.
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

#### 2.2 Mapping stats with Picard

In this service, [Picard](https://broadinstitute.github.io/picard/) is used to gather mapping quality metrics into a single table that can be easily inspected.

- For this task, start by moving to the folder: `cd 99-stats`
- Load the modules and execute the lablog: `module load picard; bash lablog`. This lablog will generate two scripts:
  - `_01_picardHsMetrics.sh` runs picard to analyze the `sample.cram` files from sarek's results and output [multiple quality metrics](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics).
  - `_02_hsMetrics_all.sh` filters the output from the previous step, keeping only the most relevant columns regarding mapping stats.

### 3. Post-processing of results from sarek

- Enter the post-processing folder and execute the lablog: `cd 02-postprocessing; bash lablog`
- Load the corresponding modules that can be found in the lablog: `module load GATK/X.X.X...`
- Sequentially execute each of the scripts previously generated to filter the results from Sarek's variant calling (vcf):
  - `_01_separate_snps_indels.sh` separates SNPs and indels into two different vcf files: `snps.vcf.gz` & `indels.vcf.gz`
  - `_02_filter.sh` apply several filters to process the vcf files.
  - `_03_merge_vcfs.sh` merges the two previous vcf files and `_04_gzip.sh` compresses the merged vcf.
  - If everything went well, you should find a file named `variants_fil.vcf.gz`, which contains the merged filtered variants.

### 4. Annotation

This step is performed apart from sarek because we aim to provide a more thorough annotation of the variants, which will include prediction of effect and correlation with the disease thanks to [Ensembl's Variant Effect Predictor (VEP)](https://www.ensembl.org/info/docs/tools/vep/index.html) and [exomiser](https://exomiser.readthedocs.io/en/latest/advanced_analysis.html). This latter will also include inheritance typing.

- Move to the folder: `cd 03-annotation`. Which will contain the following files:
- Before executing the lablog, you might need to modify exomiser's configuration file (`exomiser_configfile.yml`):
  - In the case of **exomeEB**, the researcher should have included a list of genes to target. These genes should be included in the line containing `genePanelFilter`. It should look like this `genePanelFilter: {geneSymbols: ['gene1', 'gene2' ...]}`
  - For both **exomeEB** and **exometrio**, make sure that the bed files are correctly located in `REFERENCES/`: e.g. `Illumine_Exome_CEX_TargetedRegions.bed` as this is the variable going into the line `intervalFilter: {bed: BED_FILE}`
  - When running both **wgstrio** and **exometrio** services, make sure that the `family.ped` pedigree file is correctly generated, as it is essential to perform inheritance mode prediction.
- Now you can run the lablog: `bash lablog`. Double-check that the previous variables are correctly located in the configuration file.
- Load the modules as indicated in the lablog: `module load BCFtools/X.X.X VEP/X.X.X R/X.X.X Java/X.X.X`

We are ready to initialize the annotation process:

- `bash 01_run_bcftools_query` will execute `_01_bcftools_query`, using awk and BCFtools [query](https://samtools.github.io/bcftools/howtos/query.html) to modify the VCF of variants created in 02-postprocessing (`variants.fil_mod.vcf`) and will create a table of variants inside vep folder: `vep/variants.table`.
- `_02_vep_annotation.sh` will annotate all variants from the exome/WGS analysis (takes `variants.fil_mod.vcf` and creates `vep_annot.vcf`)
- `_03_vep_plugin_NSFP_parse.sh` calls the auxiliar script `Merge_All.R` which edits `vep_annot.vcf` and paste the columns from VEP-plugin (located in /vep/variants.table) and the genic data from the [dbNSFP database](http://database.liulab.science/dbNSFP), generating a bigger table (`variants_annot_all.tab`)
  - For ExomeEB, this script is separated into 3 scripts:
    - `_03_merge_data1.sh` creates a table `/vep/vep_dbNSFP.txt` with the genic data from dbNSFP database.
    - `_04_merge_data2.sh` merges the data from the previous table with the `variants.table` from sarek vcf, creating `variants_annot_all.tab`.
    - `_05_conseq_filtering.sh` filters variants that are moderately and highly present, creating `variants_annot_highmoderate.tab`.
- The next step is running exomiser, which differs between the pipelines:
