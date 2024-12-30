# ExomeEB-ExomeTrio-WGSTrio (sarek)

These services are grouped together because they share most of the steps needed for the analysis. Nonetheless, they are aimed at different goals:
- **ExomeEB** stands for Exome in a single patient of [**Epidermolysis bullosa**](https://www.mayoclinic.org/diseases-conditions/epidermolysis-bullosa/symptoms-causes/syc-20361062).
- **ExomeTrio** is an **Exome** analysis from 2 (or more) related patients (usually Father, Mother and Children), taking **inheritance** into account.
- **WGSTrio** is a **Whole Genome Analysis** of 2 (or more) related patients (usually Father, Mother and Children), taking **inheritance** into account.

All these services use [**nf-core/sarek**](https://nf-co.re/sarek/), a pipeline for **mapping**, **variant Calling** and **annotation** of either **Whole Genome** or **Targeted** sequencing data, specially designed for Human and Mouse (although it can work with any reference genome).

## Step-by-step guide

### 1. Create the service using the [buisciii tools](https://github.com/BU-ISCIII/buisciii-tools).

- Create the service from the local terminal using the corresponding [**iSkyLIMS**](https://iskylims.isciii.es/) Resolution ID (e.g. `bu-isciii --log-file SRVIIERXXX.X.tool.log new-service SRVIIERXXX.X`).

  >[!NOTE]
  >The option `--log-file` will save a log file for tracking purposes in a specific location. **This option should be used every time the BU-ISCIII tool is used for the service**. For instance, you may want to name the log as `SRVIIERXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVIIERXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVIIERXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results) file.<br>
  >
  > When being asked whether you want to skip or not the service folder creation, type `N` to create the template folder, and then select the corresponding template folder (either `exometrio`, `exomeeb` or `wgstrio`).

- A folder will be created within `/data/bi/services_and_colaborations/IIER/human_genetics` with the full name of the service, with the following format (YYYYMMDD = YearMonthDay):
  - For **ExomeEB**: `SRVIIERXXX_YYYYMMDD_EXOMAEBXXX_researcheruser_S`
  - For **ExomeTrio**: `SRVIIERXXX_YYYYMMDD_TRIOXXX_researcheruser_S`
  - For **WGSTrio**: `SRVIIERXXX_YYYYMMDD_WGSTRIOXXX_researcheruser_S`
- Go to the recently created folder. Check that the number of files matches the number of samples that was specified in the service in iSkyLIMS (the number of files should be the number of samples x 2 if there are paired-end reads, since there is a file of forward reads and one of reverse reads).
  ```
  cd SRVIIERXXX_YYYYMMDD_WGSTRIOXXX_researcheruser_S/RAW
  ll *.fastq.gz | wc -l
  ```
- For **exome** services, the `REFERENCES` folder should include BED files with coordinates for the targeted regions, exons and/or genes during sequencing.
- If everything is alright, move to the `ANALYSIS` folder and execute the `lablog` file (this `lablog` might be named after the name of the template e.g. `lablog_exometrio`).
  - This first `lablog` will rename the main ANALYSIS folder (`DATE_ANALYSIS01`) to the current date and create the folder `00-reads` with symlinks to the .fastq files stored in `RAW`.
- Finally, move the folder to the computing resource using `bu-isciii --log-file SRVIIERXXX.X.tool.log scratch --direction service_to_scratch SRVIIERXXX.X`. Use the specific option you are using to name the log (i.e. `SRVIIERXXX.X.service_to_scratch.log`).
- From now on, all the analyses must be executed from the folder located within `scratch`.

### 2. Analysis starts: Sarek

- Execute the `lablog` file stored inside `DATE_ANALYSIS01_SERVICE`. This `lablog` will:
  - Make symbolic links to the `00-reads` folder and the file called `samples_id.txt`.
  - Create `samplesheet.csv`, containing the sample names necessary for sarek execution. Please make sure this file follows the [**required structure**](https://nf-co.re/sarek/3.5.0/docs/usage/#input-sample-sheet-configurations), since this is the input file used by sarek.
  - Create `sarek.sbatch`, the script with default configuration to run sarek, which is launched with `_01_run_sarek.sh`. Make sure everything looks correct, and check that you have `samplesheet.csv` and the corresponding .config file in `DOC`.
  - Create `../../DOC/family.ped` a file containing the pedigree of the samples in [PED format](https://gatk.broadinstitute.org/hc/en-us/articles/360035531972-PED-Pedigree-format).
     - **You should check the `family.ped` file when there are more than 3 samples in the case of trios, as it might need some editing.**

- As indicated in the `lablog` file, before running `_01_run_sarek.sh`, you need to load the required modules: `module load Nextflow singularity`
- Then, you can run `bash _01_run_sarek`.
- Once sarek is finished (**check the .log file before continuing**), you can run `_02_clean.sh`, an auxiliar script that removes folders generated during the workflow which are not used in further steps of the analysis (`work` and `01-sarek/gatk4/`, specifically).
- While this is happening, you can continue with the service. Please go to the `99-stats` folder.

#### 2.2 Mapping stats with Picard.

In this step of the service, [**Picard**](https://broadinstitute.github.io/picard/) is used to gather mapping quality metrics into a single table that can be easily inspected.
- For this task, start by moving to the folder: `cd 99-stats`
- Load the singularity module (if you haven't yet) and execute the `lablog`: `module load singularity; bash lablog`. This lablog will generate two scripts:
  - `_01_picardHsMetrics.sh` runs **picard** to analyze the `sample.cram` files from sarek's results and then output [**multiple quality metrics**](http://broadinstitute.github.io/picard/picard-metric-definitions.html#HsMetrics).
  - ` _02_hsMetrics_all.sh` filters the output from the previous step, keeping only the most relevant columns regarding mapping stats.
  
While picard runs, you can keep going with the rest of the service.

### 3. Post-processing of results from sarek.

- Enter the post-processing folder and execute the `lablog`: `cd 02-postprocessing; bash lablog`, which will create several scripts.
- If you have already loaded the singularity module, execute sequentially each one of the scripts previously generated to filter the results from Sarek's variant calling (vcf):
  - `_01_separate_snps_indels.sh` uses `gatk SelectVariants` to separate SNPs and indels into two different vcf files: `snps.vcf.gz` & `indels.vcf.gz`
  - `_02_filter.sh` uses `gatk VariantFiltration` to apply several filters to process the vcf files.
  - `_03_merge_vcfs.sh` uses `gatk MergeVcfs` to merge the two previous vcf files.
  - `_04_gzip.sh` uses gzip to compress the merged vcf file.
- If everything went well, you should find a file named `variants_fil.vcf.gz`, which contains the merged filtered variants. You can then go to the `03-annotation` folder.

### 4. Annotation.

This step is performed apart from sarek because we aim to provide a more thorough annotation of the variants, which will include now a prediction of effect and correlation with the disease thanks to [**Ensembl's Variant Effect Predictor (VEP)**](https://www.ensembl.org/info/docs/tools/vep/index.html) and [**exomiser**](https://exomiser.readthedocs.io/en/latest/advanced_analysis.html). This latter will also include inheritance mode.

>[!WARNING]
>For the following steps, some auxiliar R scripts will be employed. Please make sure you load the necessary modules:
>```
>module load singularity R
>```
>Furthermore, please make sure you have correctly set up your environment to run all the necessary jobs properly. Check the [**Usage**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Usage) page in the Wiki.

- Move to the corresponding folder: `cd 03-annotation`. This folder should contain the following files:
  - **In all cases**:
    - `exomiser_configfile.yml`: a configuration file used by exomiser.
    - `lablog`: a file that, when being executed, will create several scripts, whose utility will be explained below.
  - Plus:
    - **ExomeEB**:
      - `merge_dbNSFP.R` and `merge_parse.R`: auxiliar R scripts used by the scripts created by the `lablog` file.
    - **ExomeTrio**:
      - `Merge_All.R`: auxiliar R script used by the scripts generated by the `lablog` file.
      - `header_vep_final_annot.txt`: auxiliar .txt file composed of one line consisting in a set of column names that will be used by the `lablog` file.
      - `inheritance_types.txt`: auxiliar .txt file that will be used by the `lablog` file, composed of 5 lines, which mean:
        - `AD`: Autosomic dominant
        - `AR`: Autosomic recessive
        - `MT`: Mitochondrial
        - `XD`: X-linked dominant
        - `XR`: X-linked recessive
    - **WGSTrio**:
      - `Merge_All.R`: auxiliar R script used by the scripts generated by the `lablog` file.
      - `exomiser_configfile_exome.yml` and `exomiser_configfile_genes.yml`: configuration files used by exomiser, apart from `exomiser_configfile.yml`.
      - `header_vep_final_annot.txt`: auxiliar .txt file composed of one line consisting in a set of column names that will be used by the `lablog` file.
      - `inheritance_types.txt`: auxiliar .txt file that will be used by the `lablog` file, composed of 5 lines, which mean:
        - `AD`: Autosomic dominant
        - `AR`: Autosomic recessive
        - `MT`: Mitochondrial
        - `XD`: X-linked dominant
        - `XR`: X-linked recessive
- Before executing the `lablog` file, **you might need to modify exomiser's configuration file/s** (`exomiser_configfile.yml`):
  - In the case of **exomeEB**, the researcher should have included a list of genes to target in the request message. These genes should be included in the line containing `genePanelFilter`. It should look like this `genePanelFilter: {geneSymbols: ['gene1', 'gene2' ...]}`. There is already a list of genes included in these configuration files since these are usually the ones the researchers are interested in, but **please check that this list is still correct in your particular case**.
  - For **exometrio**, make sure that the .bed files are correctly located in `REFERENCES/`: e.g. `Illumine_Exome_CEX_TargetedRegions.bed` as this is the variable going into the line `intervalFilter: {bed: BED_FILE}`
  - When running both **wgstrio** and **exometrio** services, make sure that the `family.ped` pedigree file is correctly generated, as it is essential to perform inheritance mode prediction.
- Now you can run the lablog: `bash lablog`. Double-check that the previous variables are correctly located in the configuration file.
- Make sure you have already loaded all the necessary modules as stated before.

We are ready to initialize the **annotation** process. Run these scripts sequentially:
- `01_run_bcftools_query.sh` will execute `aux_01_bcftools_query`, using `awk` and [`BCFtools query`](https://samtools.github.io/bcftools/howtos/query.html) to modify the VCF files of variants created in `02-postprocessing` (specifically, the `variants_fil.vcf` file), and will create a table of variants inside the `vep` folder: `vep/variants.table`.
- `_02_vep_annotation.sh` will annotate all variants from the exome/WGS analysis (takes `variants_fil_mod.vcf` (created previously after running `01_run_bcftools_query.sh`) and creates `vep_annot.vcf`).
- `_03_Vep_plugin_dbNSFP_parse.sh` calls the auxiliar script `Merge_All.R`, which edits `vep_annot.vcf` and pastes into it the columns from VEP-plugin (located in `/vep/variants.table`) and the genic data from the [dbNSFP database](http://database.liulab.science/dbNSFP), generating a bigger table called `variants_annot_all.tab`. This table is then filtered using `awk` to keep only low frequency variants in another file called `variants_annot_filterAF_head.tab`, thanks to the auxiliary script called `aux_03_awk.sh`.
  - For **ExomeEB**, **this script is split into 3 separated scripts**: 
    - `_03_merge_data1.sh` creates a table called `/vep/vep_dbNSFP.txt` with the genic data from dbNSFP database, thanks to the `merge_dbNSFP.R` script.
    - `_04_merge_data2.sh` merges the data from the previous table with the `variants.table` file from sarek vcf, creating `variants_annot_all.tab` thanks to the `merge_parse.R` script.
    - `_05_conseq_filtering.sh` filters only for variants that are moderately and highly present, creating `variants_annot_highModerate.tab`.
- The next step is running **exomiser** (`_04_exomiser_*.sh` for **exometrio** and **wgstrio**, or `_06_exomiser_exome.sh` for **exomeEB**). In all cases, before running these scripts, **please make sure that the variable `proband`**, which initially has associated the value `PROBAND`, **has correctly been substituted by the name of the sample being used as proband** (in the case of **exomeEB**, it is the sample that is being analysed itself; in the case of **exometrio** and **wgstrio**, the researcher should indicate in the request message the name of the proband). For example, if the proband name is `EB0001`, after running `lablog`, you should have a line saying `proband: EB0001_EB0001`. The process differs between the pipelines:
  - In **exomeEB**, exomiser will target relevant genes for the disease and will not offer inheritance mode.
  - For **exometrio** and **WGStrio**, it won't target specific genes, but it will use the `DOC/family.ped` file to split variants depending on the inheritance mode.
- Finally:
  - For **ExomeEB**, run the `_07_gzip_table.sh` script to compress the `variants_annot_all.tab file`.
  - For **trio** services, run `_05_filter_inheritance_*.sh`, which splits the annotated variants in `/variants_annot_all.tab` depending on the mode of inheritance into multiple files called `vep_annot_**.txt`.

Once the service is finished, you should go to `RESULTS` and execute the corresponding `lablog` file. Depending on the service you ran, you may find different files, but you should always check the following ones:
- `multiqc_report.html`: for quality control.
- `mapping_metrics.csv`: to check the overall coverage and mapping quality.
- `variants_*.tab`: can be used to find the variants with the highest predicted effect by vep. It can also be used to find **low frequency variants** (e.g. **AF < 0.05|0.01**).
- `exomiser.html`: **ONLY** in **exomeEB services**, in order to select the most relevant genes found, based on **exomiser** and **variant scores**.
- `annotation_tables`.

---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools.

    $ bu-isciii --log-file SRVIIERXXX.X.finish.log finish SRVIIERXXX.X

>[!WARNING]
>When running the `finish` and the `bioinfo-doc` modules, you'll be asked which SFTP folder to copy the data to. This depends on the situation:
>* **If sample names start with ND (from Non Diagnosed), the SFTP folder will be `SpainUDP`.**
>* **If sample names DO NOT start with ND, the SFTP folder will be `GeneticDiagnosis`**. This is the case of ExomeEB services, since EB is not an undiagnosed disease.

This module will do several things. First, it **cleans up** the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space:
* For **ExomeEB**, please make sure the following are deleted:
  * `01-sarek/preprocessing/markduplicates`
  * `02-postprocessing`
  * `03-annotation/dbNSFP_ENSG_plugin_hg19.txt`
  * `03-annotation/dbNSFP_ENSG_plugin_Columns.txt`
* For **ExomeTrio** and **WGSTrio**, please make sure the following are deleted:
  * `01-sarek/preprocessing/markduplicates`
  * `02-postprocessing`
  * `03-annotation/vep`
  * `03-annotation/exomiser`
  * `03-annotation/dbNSFP_ENSG_plugin_hg19.txt`
  * `03-annotation/dbNSFP_ENSG_plugin_Columns.txt`
  
Then, it copies all the service files back to its `/data/bi/services_and_colaborations/IIER/human_genetics/` folder, and also copies the content of this service to the researcher's SFTP repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/bioinfo_doc/` folder.

    $ bu-isciii --log-file SRVIIERXXX.X.tool.log bioinfo-doc SRVIIERXXX.X

Remember to save the logs with the corresponding name (i.e. `SRVIIERXXX.X.service_info.log` or `SRVIIERXXX.X.delivery.log`).

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

Lastly, once the service has been delivered and the e-mail has been sent, remember to remove all the files related to this service from `scratch_tmp`:

    $ bu-isciii --log-file SRVIIERXXX.X.tool.log scratch SRVIIERXXX.X
    $ remove_scratch

## ExomeEB-ExomeTrio-WGSTrio report template

After copying the service to the corresponding SFTP folder, and before delivering the service, please fill in this template with all the relevant information and wait for the delivery approval.

>[!NOTE]
>You can get the instrument name from [**iSkyLIMS**](https://iskylims.isciii.es/), by checking the run name associated with the samples of the service.

### ExomeEB

***SRVIIERXXX_XXXXXXXX_EXOMAEBXX_XXXXXXXXX***
* **Servicio**: Análisis de exoma con panel de genes de la muestra: EBXXXX
* **Número de muestras**: 1
* **Estado**: Finalizado, copiado al SFTP
* **Instrumento y longitud**: XXXXX (XxXXX)
* **Cantidad de lecturas**: XXXXX
* **Calidad general**: XXXXX
  * Profundidad media / porcentaje target reads a 10X: XXXX.XX / XX.XX%
* **Resultados generales**: Los genes con exomiser score > 0.05 y variant score > 0.1 son: XXXXX
* **Incidencias muestras individuales**: indicate any incidences that you might consider relevant for the researcher.

>[!NOTE]
> * The **average depth** can be obtained from the `MEAN TARGET COVERAGE` column from `mapping_metrics.csv` from the folder `RESULTS`.
> * The **percentage of 10X target reads** can be obtained from the `PCT TARGET BASES 10X` column from `mapping_metrics.csv` from the folder `RESULTS`.
> * To see which genes have an **exomiser score > 0.05** and a **variant score > 0.1** (both conditions have to be met), please check the `exomiser.html` file from `RESULTS`.

### ExomeTrio

***Exoma TRIO SRVIIERXXX_XXXXXXXX_TRIOXX_XXXXXXXXX***

* **Servicio**: Análisis de trío de exomas con las muestras: NDXXXX, NDXXXX, NDXXXX
* **Número de muestras**: 3
* **Estado**: Finalizado y copiado al SFTP
* **Instrumento y longitud**: XXXXX (XxXXX)
* **Cantidad de lecturas**: XXXXX - XXXXX
* **Calidad general**: XXXXX
  * **Cobertura (profundidad) media**: NDXXXX: XXXX.XX / NDXXXX: XXXX.XX / NDXXXX: XXXX.XX
  * **Porcentaje de target reads a 10X**: NDXXXX: XX.XX% / NDXXXX: XX.XX% / NDXXXX: XX.XX%
* **Incidencias muestras individuales**: indicate any incidences that you might consider relevant for the researcher.

>[!NOTE]
> * The **average depth** can be obtained from the `MEAN TARGET COVERAGE` column from `mapping_metrics.csv` from the folder `RESULTS`.
> * The **percentage of 10X target reads** can be obtained from the `PCT TARGET BASES 10X` column from `mapping_metrics.csv` from the folder `RESULTS`.

### WGSTrio

***SRVIIERXXX_XXXXXXXX_WGSTRIOXX_XXXXXXXXX***

* **Servicio**: Análisis de trío WGS con las muestras: NDXXXX, NDXXXX, NDXXXX
* **Número de muestras**: 3
* **Estado**: Finalizado y copiado al SFTP
* **Instrumento y longitud**: XXXXX (XxXXX)
* **Cantidad de lecturas**: XXXXX - XXXXX
* **Calidad general**: XXXXX
  * **Cobertura (profundidad) media**: NDXXXX: XXXX.XX / NDXXXX: XXXX.XX / NDXXXX: XXXX.XX
  * **Porcentaje de target reads a 10X**: NDXXXX: XX.XX% / NDXXXX: XX.XX% / NDXXXX: XX.XX%
* **Incidencias muestras individuales**: indicate any incidences that you might consider relevant for the researcher.

>[!NOTE]
> * The **average depth** can be obtained from the `MEAN TARGET COVERAGE` column from `mapping_metrics.csv` from the folder `RESULTS`.
> * The **percentage of 10X target reads** can be obtained from the `PCT TARGET BASES 10X` column from `mapping_metrics.csv` from the folder `RESULTS`.