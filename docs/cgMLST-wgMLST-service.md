# How to perform a cgMLST/wgMLST service

## Introduction

This is a brief tutorial on how to perform a **cgMLST/wgMLST** service as a member of the ISCIII's Bioinformatics Unit (BU-ISCIII)! First of all, please remember that cgMLST/wgMLST is associated with the **_Bacteria: Core genome or whole genome Multi-Locus Sequence Typing analysis (cg/wgMLST)_** service from the [**service catalog**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Service-portfolio).

**MLST** stands for **MultiLocus Sequence Typing**. For each hosekeeping gene, the different sequences present within a bacteria species are assigned as distinct alleles and, for each isolate, the alleles for each of the loci define the allelic profile or **Sequence Type** (**ST**). Do not confuse with SeroType, they are not the same.

Each bacterial species has their own **MLST schema**. A **Schema** is a **pre-defined set of loci that is used in MLST analyses**. Traditional MLST schemas relied in **7 loci** that were internal fragments of housekeeping genes. In genomic analyses, schemas can be:

- **cgMLST** (**core-genome MLST**): Set of loci that are present in the **majority** of **strains** for **core genome** (cg) MLST schemas, typically a threshold of presence in **95%** of the strains is used in Schema Creation.
- **wgMLST** (**whole-genome MLST**): Set of loci that are present in **at least one of the analyzed strains** in the Schema Creation for **whole genome MLST schemas**.
- **pgMLST** (**pan-genome MLST**): Set of loci that are present in **at least one of the analyzed strains** in the Schema Creation for **pan genome MLST schemas**.
- **agMLST** (**accesoty-genome MLST**): Set of loci that are present in **less than 95% of the strains for accessory genome MLST schemas**.

Let's get started with the service. When performing a cgMLST/wgMLST service, remember that **this service des not have a specific associated acronym**. Usually, cgMLST/wgMLST service is part of assembly service (see the [**Assembly documentation**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service)), because it needs the assemblies to be performed.

Additionally, it is usually performed along with the [**Characterization**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/Characterization-service.md), [**PlasmidID**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/plasmidID-service.md) and [**Snippy**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) services, so the acronym will probably be one as explained in the wiki pages of these services.

### Software used

For this service, the main tool that is used is **[chewBBACA](https://chewbbaca.readthedocs.io/en/latest/index.html)**. This software is used for the **creation** and **evaluation** of **core genome** and **whole genome MultiLocus Sequence Typing (cg/wgMLST) schemas and results**.

According to its manual, chewBBACA allows to define the target loci in a schema based on multiple genomes (e.g. define target loci based on the distinct loci identified in a dataset of high-quality genomes for a species or lineage of interest) and performs allele calling to determine the allelic profiles of bacterial strains.

chewBBACA can annotate the schema loci, compute the set of loci that constitute the core genome for a given dataset, and generate interactive reports for schema and allele calling results evaluation to enable an intuitive analysis of the results in surveillance and outbreak detection settings or population studies.

Since the cg/wgMLST definition is usually done along with the construction of a phylogenetic tree, this service is usually done along with [**IQTREE**](http://www.iqtree.org/) and [**grapetree**](https://github.com/achtman-lab/GrapeTree). The usage of **IQTREE** is explained in the [**Snippy wiki**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page, while the usage of **grapetree** for the creation and visualisation of a **minimum spanning tree** will be explained here.

## Bioinformatics procedure

>[!WARNING]
>As stated before, this cg/wgMLST service is usually not done alone, but along with *de novo* assembly, characterization, snippy and PlasmidID. Therefore, you'll most likely already have created the service folder, which will still have the acronym associated to the **Characterization** service, as indicated below.

>[!NOTE]
> **Ignore the following if the service folder has already been created** (which is the most probable thing):<br><br>
>Taking the previous information into account, the first thing we need to do is take the service and click on _**Add resolution**_ in **[iSkyLIMS](https://iskylims.isciii.es/)** after logging in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym.
>
>In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:
>
>```shell
>cd /data/ucct/bi/services_and_colaborations/CNM/bacteriology
>ll -tr
>```
>
>Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered.
>
>At this point, you'll have to consider what kind of service has been requested in this case. In general, follow these guidelines:
>* If all samples correspond to a specific bacterial species, within the context of an outbreak, the service acronym should have the following structure: `SPECIESOUTBREAKXXX`. For example, if the outbreak corresponds to *Bacillus cereus* and this is the first outbreak we analyse in relation to this species, the service should be called `BCEREUSOUTBREAK001`.
>* If the service is related to one or several species, but it is not within the context of an outbreak, the service should be called after the following structure: `WGSORGANISMXX`. For example, if we have samples from several species from the same genus (like *Burkholderia*), we should call the service `WGSBURKHOLDERIA01` (if it's the first service related to *Burkholderia* species).
>* If the service corresponds to samples from several species, not necessarily from the same genus, and therefore not all species can be categorised inside one name, we should call the service after the following structure: `CHARACTERIZATIONXX`. For example, if our service includes samples with a wide variety of species from different genera, we should call the service `CHARACTERIZATION01` (if it's the first service related to multiple species).
>
>Regardless of the acronym that has been assigned to the service, your service acronym will be the last service's number + 1. For example, if you have chosen the `CHARACTERIZATION` acronym for your service and the last folder after executing `ll -tr` is `SRVCNM1200_20240823_CHARACTERIZATION07_svaldezate_S`, the service acronym for your new service will be _**CHARACTERIZATION08**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools micromamba environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

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

* `Do you want to skip folder creation?`:
  * **If the service folder has not been created yet in the `services_and_collaborations` folder**, answer **NO**.
  * **If this is not the first resolution associated with the service or another service has already been performed for the current resolution**, answer **YES**.

* Next, specify `wgmlst_chewbbaca`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can then get into the `ANALYSIS` folder and we'll find, apart from folders and files from other services done previously, the following items inside:
- `lablog_chewbbaca`: an executable file that creates the `00-reads` folder, moves inside, creates symbolic links to the reads renaming them and renames the `ANALYSIS01_CHEWBBACA` folder to `DATE_ANALYSISXX_CHEWBBACA`.
- `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.
- `ANALYSIS01_CHEWBBACA`: a folder with the main cgMLST/wgMLST analysis files.

First of all, let's **check in the `lablog_chewbbaca` if the renaming of the `ANALYSIS01_CHEWBBACA` is correct**:

- If cgMLST/wgMLST analysis is the only analysis in the service, the folder will be `DATE_ANALYSIS01_CHEWBBACA`.
- If cgMLST/wgMLST analysis is part of other services, you will have to sum as many numbers as other analyses you have done previously.

Now we can execute the `lablog_chewbbaca` file:

```bash
bash lablog_chewbbaca
```

Once this file has been executed, please take into consideration that this service is usually performed along with other pipelines, so **run all the necessary `lablog` files before moving on to the next BU-ISCIII module**.

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the analysis. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

```shell
buisciii scratch SRVCNMXXX.X
```

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S/ANALYSIS/DATE_ANALYSIS0X_CHEWBBACA
```

> [!WARNING]
> Please note that the chewBBACA service is usually performed along with other services; that's why this folder is called `ANALYSIS0X`. During the previous assembly service, **we should have saved the trimmed sequences**, since **they will be needed** for this service.

Once we're at this point, and before executing anything else, we should load all the necessary dependencies:

```shell
module load singularity
```

If everything is OK, and before running any analyses, we should first prepare the schema that will be used by chewBBACA for allele calling.

### Create training file

The first step for chewBBACA is creating a **training file** with **Prodigal**; this is strongly recommended by the chewBBACA developers. Since chewBBACA uses Prodigal to define the CDSs, a very important step to ensure the reproducibility of the allele calls, specially in what concerns the definition of the start codon for an allele, is the use of a Prodigal training file. Without this training file, Prodigal uses the provided .fasta files to do a training run prior to the analysis, and therefore, different assemblies may lead to slightly different HMM training procedures that will, in turn, return alleles that may vary in the start codon.

In the service's `REFERENCE` folder, please run:

```bash
singularity exec prodigal:2.6.3--h7b50bb2_10 prodigal -i ./<reference_NCBI_genome>.fasta -t <output_training_file>.trn -p train
```

You'll obtain a **.trn** file that you should indicate when running all the necessary chewBBACA functions, by means of the `-ptf` function.

### Schema preparation

We have different ways to obtain the schema: to **create** it or to **download** it. **It is better to download the schema from the DDBB rather than creating one**, obviously if the schema is suitable for our situation.

#### Download an already existing schema

There are different websites from which to download a schema for a specific species:

- [**Ridom schemas**](https://www.cgmlst.org/ncs/): Used by **SeqSphere**.
  - **cgMLST** schemas: if there is a **cgMLST** schema for your species of interest, click on its schema > **Show Targets** > **Download alleles as FASTA**. A .zip file containing several .fasta files will be downloaded.
  - The downloaded schema will have to be later processed by chewBBACA in order to make it valid for its usage, by means of the `PrepExternalSchema` function from chewBBACA.
- [**chewBBACA**](https://chewbbaca.online/stats):
  - **wgMLST** schemas: Find the bacteria you want and select **`SCHEMA DETAILS`** > Go to **"Compressed Schema"** and select **`DOWNLOAD`**
  - These schemas **do not** need to go through `PrepExternalSchema` from chewBBACA.
- [**pubMLST**](https://pubmlst.org/): example for *Streptoccocus pnemoniae*:
  - **ST 7 genes**: pubMLST > Organisms > *Streptoccocus pneumoniae* > **Typing** > **Downloads** (on the right) > **Allele Sequences** > **Typing** > **MLST** (Download the 7 loci one by one), then merge them into a single .fasta file.
  - **ST profile definitions**: **pubMLST** > **Organisms** > *Streptoccocus pneumoniae* > **Typing** > **Downloads** (right) > **MLST/Allelic profiles** > **MLST** (download button 2nd column) > **CTRL + A** > **Copy** and **paste** to file.
  - **cgMLST**: Use the `get_files_from_rest_api.py` script from [**download_bigsdb_api**](https://github.com/BU-ISCIII/download_bigsdb_api) this way:
    - Let's say you want to download a cgMLST schema from *Yersinia pestis*. In the service's `REFERENCE` folder where you want to download the files, run:
    ```
    python3 /data/ucct/bi/pipelines/download-bigsdb-api/get_files_from_rest_api.py --output_dir . schema -api pasteur_yersinia -sch "Yersinia cgMLST"
    ```
    - For this to work correctly, please take into account that the variable `pasteur_yersinia` that is indicated in the `-api `option must already be defined in the code of `get_files_from_rest_api.py`, corresponding to the **URL of the schema**. The `-sch` option must also be indicated, which corresponds to the **description** of the **schema** we want to download. **This might differ between organisms and databases.**
    - For example, if the URL is https://bigsdb.pasteur.fr/api/db/pubmlst_yersinia_seqdef/schemes, we'll see this in our browser:
      ```
      {"records":4,"schemes":[{"scheme":"https://bigsdb.pasteur.fr/api/db/pubmlst_yersinia_seqdef/schemes/4","description":"Virulence factors"},{"scheme":"https://bigsdb.pasteur.fr/api/db/pubmlst_yersinia_seqdef/schemes/2","description":"Y.enterocolitica cgMLST"},{"description":"Yersinia cgMLST","scheme":"https://bigsdb.pasteur.fr/api/db/pubmlst_yersinia_seqdef/schemes/1"},{"description":"Y.pseudotuberculosis cgMLST","scheme":"https://bigsdb.pasteur.fr/api/db/pubmlst_yersinia_seqdef/schemes/3"}]}
      ```
      We must write "Yersinia cgMLST" in the `-sch` option since this is the description of the schema we want to download.
    - After running the previous command, the download will start and multiple .fasta files will be downloaded.

- [**Enterobase**](https://enterobase.warwick.ac.uk/)

Once you have downloaded the schema of interest (**as .fasta files stored within one single subfolder**), you might have to **prepare** it so that it can be used by chewBBACA for allele calling.

This preparation procedure enables the adaptation of external schemas so that it is possible to use those schemas when running chewBBACA. External schemas are defined with specific parameters that might differ from the parameters and conditions enforced by chewBBACA. Therefore, these external schemas need to be processed to filter out sequences that do not meet a set of criteria applied to create every chewBBACA schema.

To prepare an external schema, when running the `lablog` file from the `02-chewbbaca` folder, you'll have to indicate that the schema is not ready, so that several scripts are created in order to prepare the schema for chewBBACA and perform the allele calling. You'll see this later in this guide.

#### Creation of a custom schema

 If you don't have a proper schema in the public repositories listed previously or you cannot use any of them for some reason, you must then **create** a **custom schema**, for which you can use **chewBBACA**.

[**chewBBACA**](https://chewbbaca.readthedocs.io/en/latest/index.html), as indicated before, is a comprehensive pipeline for the creation and validation of whole genome and core genome MultiLocus Sequence Typing (wg/cgMLST) schemas, providing an allele calling algorithm based on [BLAST Score Ratio](http://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-6-2) that can be run in multiprocessor settings, as well as a whole set of functions to visualise and evaluate allele variation in the loci. chewBBACA performs the schema creation and allele calling procedures on complete or draft genomes resulting from *de novo* assemblies.

For the creation of a custom schema, we should first go to the `REFERENCES` folder of the service. Let's say you have a set of genomes that have already been downloaded inside a subfolder from `REFERENCES` in .fasta format. For the creation of the schema, we should run the `lablog_chewbbaca` file from `REFERENCES`, which will create two scripts:
* `_01_create_schema.sh`: when running this script, the `CreateSchema` from chewBBACA will be run, creating a new schema using the genomes that were previously downloaded as input. You'll have to indicate the absolute path of this subfolder in the terminal. The output is stored in a subfolder called `created_schema`, which will store at the same time another folder called `schema_seed`, which will be used for allele calling subsequently.
* `_02_extract_cgmlst.sh`: this script runs the `ExtractCgMLST` function from chewBBACA to extract the cgMLST scheme after perfoming the allele calling, so **DO NOT run this script until the allele calling procedure has been done**.

### Analysis

Once we have the schema ready (either downloaded or either created), we should go into the `DATE_ANALYSIS0X_CHEWBBACA` folder associated with our service. Once we're inside, we will see the following folders/files related with cgMLST/wgMLST:

- `lablog`: this script will create symbolic links to `00-reads` and the `samples_id.txt` file, and it will create the `03-grapetree` and `01-assemblies` folders. It will also move inside the `01-assemblies` folder, copy and decompress the _de novo_ assemblies previously generated when running Unicycler (please check the [**Assembly documentation**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service)).
- `02-chewbbaca`: this folder will contain the scripts for chewBBACA analysis.

Let's execute the `lablog` file:

```bash
bash lablog
```
>[!NOTE]
>**BEAR IN MIND**: please **select those samples that correspond to the bacteria we want to analyse** and **exclude** those that:
>- were not reconstructed.
>- were contaminated.
>- were other bacteria.

#### `02-chewbbaca`

Check that all files and folders are linked/copied/created properly and move inside `02-chewbbaca` folder. There, you'll see a `lablog` file which will ask you if your schema is already prepared for chewBBACA. It is a yes/no question. The .sh files generated by the `lablog` will depend on your answer to this question:

- If you answer **YES**, there is no need to make chewBBACA prepare the schema and evaluate it, so **it will only create the scripts to perform the allele calling and allele calling evaluation**. Therefore, you'll find two scripts after running `lablog`, which should be run sequentially:
  - `_01_chewbbaca_calling.sh`: this script will run the `AlleleCall` function from chewBBACA to perform allele calling, using the assemblies stored within `01-assemblies` as input and the schema that is already ready (if you already ran the `PrepExternalSchema` function, you should have the adapted schema in a folder called `prep_schema`; if you created a custom schema, the schema to indicate here should be the `schema_seed` subfolder that was mentioned previously). The output will be stored in a subfolder called `allele_calling`.
  - `_02_chewbacca_allelecall_evaluator.sh`: this script will run the `AlleleCallEvaluator` function from chewBBACA, using the `allele_calling` folder as input and the same schema that was indicated before. The output will be stored in a new subfolder called `allele_calling_evaluation`.
- If you answer **NO**, it will **first** create the scripts to make chewBBACA **prepare** the **schema** and its **evaluation**, **prior to the allele calling and allele calling evaluation**. Therefore, you'll find four scripts after running `lablog`, which should be run sequentially:
  - `_01_prep_schema.sh`: this script will run the `PrepExternalSchema` from chewBBACA in order to prepare the downloaded schema for chewBBACA. This function takes the schema that was downloaded as input and generates a new subfolder called `prep_schema`, with the schema ready to be used for allele calling.
  - `_02_analyze_schema.sh`: this script will run the `SchemaEvaluator` from chewBBACA in order to evaluate the schema that was downloaded. The output is stored in a new subfolder called `analyze_schema`.
  - `_03_allele_calling.sh`: this script will run the `AlleleCall` function from chewBBACA as explained before.
  - `_04_chewbacca_allelecall_evaluator.sh`: this script will run the `AlleleCallEvaluator` function from chewBBACA as explained before.

>[!NOTE]
>In both cases, the `lablog` file will ask you for the **path** to the **schema** (please indicate an **absolute path**).

##### Allele calling

In Biology, an **allele** is a specific **sequence variant** that occurs at a given **locus**. In **chewBBACA**, an **allele** needs to be a **CDS** defined by **Prodigal**. To ensure reproducibility of the CDS prediction, **the same Prodigal training file for each bacterial species should be used and provided as input**. **Every sequence** that is **included** in the final **schema** has to **represent** a **complete coding sequence** (the first and last codons must be valid start and stop codons, the sequence length must be a multiple of 3 and cannot contain in-frame stop codons) and **contain** **no invalid or ambiguous characters** (sequences must be composed of `ATGC` only).

A **BLASTP** database is created containing all the translated CDSs identified by Prodigal in the query genome. A 100% DNA identity comparison is performed first on all the genome CDSs against each locus allele database. If an **exact match** is found an **allele identification** is attributed to the **CDS**. If not, a BLAST Score Ratio (BSR) approach is used to identify the allele. To improve computational efficiency, chewBBACA processes each locus in the schema separately for homology search, parallelising the jobs using the specified number of CPUs. Therefore, for each locus, the short list containing the more divergent alleles of each locus is queried against the BLASTP database.

<img src="https://i.imgur.com/H9pKjHQ.png" width="600" height="400" align="middle" alt="chewBBACA Allele Calling"/>

##### AlleleCallEvaluator

The `AlleleCallEvaluator` function allows users to generate an **interactive HTML** **report** to **evaluate** **allele calling results** generated by the `AlleleCall` module.

The report provides **summary statistics** to evaluate results per sample and per locus (with the possibility to provide a .tsv file with loci annotations to include on a table). The report includes components to display a **heatmap** representing the loci presence-absence matrix, a **heatmap** representing the distance matrix based on allelic differences and a Neighbor-Joining (NJ) **tree** based on the Multiple Sequence Alignment (MSA) of the core genome loci.

#### cgMLST extraction

Once you have run all the scripts from `02-chewbbaca`, it is time to extract the cgMLST from the samples that are being analysed and determine how many genes compose this cgMLST. To do so, go to the `REFERENCES` folder and run `_02_extract_cgmlst.sh`.

This script will use the `results_alleles.tsv` file obtained after performing allele calling to determine the cgMLST, rendering several .tsv and .txt files, apart from an HTML report that should have the following aspect:

![cgMLST_report](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/cgMLST_report.png)

If you place your mouse on the end of the lines, you'll see the number of genes that compose your cgMLST95, cgMLST99 and cgMLST100 considering all the genomes that were used for the schema creation. You can also check the number of loci that compose the cgMLST100 in the corresponding .log file.

#### `03-grapetree`

Now we can leave `02-chewbbaca` folder and move into **`03-grapetree`** folder to perform the **Minimun Spanning Tree** with **GrapeTree**. [GrapeTree](https://enterobase.readthedocs.io/en/latest/grapetree/grapetree-about.html) is a fully interactive, tree visualization program within EnteroBase, which supports facile manipulations of both tree layout and metadata. It generates GrapeTree figures using the **Neighbor-Joining (NJ) algorithm**, the classical minimal spanning tree algorithm (**MSTree**) similar to PhyloViz, or an improved minimal spanning tree algorithm which we call **MSTree V2**.

>[!WARNING]
>This program has to be installed **locally**, not in the HPC. It requires **Python3**. To install this, please run:

```bash
conda create -n grapetree pip
conda activate grapetree
pip install grapetree
pip install pandas
```

To launch GrapeTree, just run the `grapetree` command in your terminal. You will need to load the previosuly mentioned `allele_calling_evaluation/masked_profiles.tsv` file.

>[!WARNING]
>Before uploading the `masked_profiles.tsv` file, change the row names so they only contain the sample name, not the whole assembly .fasta name. The metadata (if any) file is uploaded in the same way as the .tsv file.

Then do the following:

- **Inputs/Outputs** > **Load Files** > Select the `masked_profiles.tsv` file in allele_calling_evaluation > **Parameters For Tree Creation** > **MSTree**
- **Tree Layout**:
  - **Node Style**
    - **Show labels**
    - **Increase Node Size**
  - **Branch Style**
    - **Show labels**
    - **Increase Font Size**
    - **Log Scale**
    - **For branches**:
      - **Longer than = 700 > Shorten**
- **Context Menu**: If you have information, add metadata in a table whith two columns:
  - Col1: sample name (same as in the TSV file)
  - Col2: classification class (tratment, control)
- **Inputs/Outputs**: (**save these files in the service's `03-grapetree` folder**)
  - Save **SVG**
  - Save **Newick Tree** (nwk)

### Results obtained after the analysis

Once you have checked that everything has finished correctly by checking the `logs` folder, you'll see these files in your folder:

- **If your schema was not from ChewBBACA**:
  - `prep_schema`: Schema files prepared for chewBBACA. The proper .log file, after running `PrepExternalSchema`, will say something like:

    ```bash
    Successfully adapted 1095/1095 loci present in the input schema.

    Finished at: 2024-04-10T10:23:58
    Took  7m 17s.
    ```

  - `analyze_schema/schema_report.html`: HTML report with the **evaluation** of the **schema** used for the analysis, after running `SchemaEvaluator` The proper .log file will say something like:

    ```bash
    Computing loci statistics...
    [====================] 100%
    Provided annotations for 0 loci in the schema.

    Results available in /path/to/results.

    Finished at: 2024-04-10T15:25:26
    Took  0m 24s.
    ```
  
- **In both cases**:
  - `allele_calling`:

    - `cds_coordinates.tsv`: Contains the coordinates (genome unique identifier, contig identifier, start position, stop position, protein identifier attributed by chewBBACA, and coding strand (chewBBACA<=3.2.0 assigns 1 to the forward strand and 0 to the reverse strand and chewBBACA>=3.3.0 assigns 1 and -1 to the forward and reverse strands, respectively)) of the CDSs identified in each genome.

    - `loci_summary_stats.tsv`: Contains the classification type counts (**EXC, INF, PLOT3, PLOT5, LOTSC, NIPH, NIPHEM, ALM, ASM, PAMA, LNF**) and the total number of classified CDSs (**non-LNF**) per locus.
  
    - `paralogous_counts.tsv`: Contains the list of paralogous loci and the number of times those loci matched a CDS that was also similar to other loci in the schema.
  
    - `results_alleles.tsv`: Contains the allelic profiles determined for the input samples. The first column has the identifiers of the genome assemblies for which the allele call was performed. The remaining columns contain the allele call data for loci present in the schema, with the column headers being the locus identifiers. The **INF-** prefix in the allelic number indicates that such allele was newly inferred in that genome, and the number following the prefix is the ID attributed to such allele. For the PLOT classification, in the allelic profile output, a locus can be classified as PLOT5 or PLOT3 depending on whether the CDS in the genome under analysis matching the schema locus is located in the 5' end or 3' end (respectively) of the contig. All other annotations are identical to what was described above.
  
    - `results_statistics.tsv`: Contains the classification type counts (EXC, INF, PLOT3, PLOT5, LOTSC, NIPH, NIPHEM, ALM, ASM, PAMA, LNF), the total number of invalid CDSs, the total number of classified CDSs (non-LNF) and the total number of predicted CDSs per genome. The column headers stand for:
      - **EXC** - **EXaCt matches** (100% DNA identity) with previously identified alleles.
      - **INF** - **INFerred new alleles** that had no exact match in the schema but are highly similar to loci in the schema. The INF- prefix in the allele identifier indicates that such allele was newly inferred in that genome, and the number following the prefix is the allele identifier attributed to such allele. Inferred alleles are added to the FASTA file of the locus they share high similarity with.
      - **LNF** - **Locus Not Found**. No alleles were found for the number of loci in the schema shown. This means that, for those loci, there were no BLAST hits or they were not within the BSR threshold for allele assignment.
      - **PLNF** - **Probable Locus Not Found**. Attributed when a locus is not found during execution modes 1, 2 and 3. Those modes do not perform the complete analysis, that is only performed in mode 4 (default), and the distinct classification indicates that a more thorough analysis might have found a match for the loci that were not found.
      - **PLOT3/PLOT5** - **Possible Locus On the Tip** of the query genome contigs (see image below). A locus is classified as PLOT when the CDS of the query genome has a BLAST hit with a known larger allele that covers the CDS sequence entirely and the unaligned regions of the larger allele exceed one of the query genome contigs ends (a locus can be classified as PLOT5 or PLOT3 depending on whether the CDS in the genome under analysis matching the schema locus is located in the 5’ end or 3’ end (respectively) of the contig). This could be an artifact caused by genome fragmentation resulting in a shorter CDS prediction by Prodigal. To avoid locus misclassification, loci in such situations are classified as PLOT.
      - **LOTSC** - A locus is classified as LOTSC when the contig of the query genome is smaller than the matched allele.
      - **NIPH** - **Non-Informative Paralogous Hit** (see image below). When ≥2 CDSs in the query genome match one locus in the schema with a BSR > 0.6, that locus is classified as NIPH. This suggests that such locus can have paralogous (or orthologous) loci in the query genome and should be removed from the analysis due to the potential uncertainty in allele assignment (for example, due to the presence of multiple copies of the same mobile genetic element (MGE) or as a consequence of gene duplication followed by pseudogenization). A high number of NIPH may also indicate a poorly assembled genome due to a high number of smaller contigs which result in partial CDS predictions. These partial CDSs may contain conserved domains that match multiple loci.
      - **NIPHEM** - similar to the NIPH classification, but specifically referring to exact matches. Whenever several CDSs from the same genome match a single or multiple alleles of the same locus with 100% DNA similarity during the first DNA sequence comparison, the NIPHEM tag is attributed.
      - **PAMA** - **PAralogous MAtch**. Attributed to CDSs that are highly similar to more than one locus. This type of classification allows the identification of groups of similar loci in the schema that are classified as paralogous loci and listed in the paralogous_counts.tsv and paralogous_loci.tsv files.
      - **ALM** - **Alleles 20% Larger than the length Mode of the distribution of the matched loci** (CDS length > (locus length mode + locus length mode * 0.2)) (see image below). This determination is based on the currently identified set of alleles for a given locus. It is important to remember that, although infrequently, the mode may change as more alleles for a given locus are called and added to a schema.
      - **ASM** - similar to ALM but for Alleles 20% Smaller than the length Mode distribution of the matched loci (CDS length < (locus length mode - locus length mode * 0.2)). As with ALMs it is important to remember that, although infrequently, the mode may change as more alleles for a given locus are called and added to a schema.
  
        ![PLOT](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/PLOT.png)
        ![NIPH-NIPHEM](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/NIPH-NIPHEM.png)
        ![ALM-ASM](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/ALM-ASM.png)

    - `invalid_cds.txt`: Contains the list of alleles predicted by Prodigal that were excluded based on the minimum sequence size value and presence of ambiguous bases.
  
    - `logging_info.txt`: Contains summary information about the allele calling process.
  
    - `paralogous_loci.tsv`: Contains the sets of paralogous loci identified per genome (genome identifier, identifiers of the paralogous loci and the coordinates of the CDS that is similar to the group of paralogous loci).

    - `results_contigsInfo.tsv`: Contains the loci coordinates in the genomes analyzed. The first column contains the identifier of the genome used in the allele calling and the other columns (with loci names in the headers) the locus coordinate information or the classification attributed by chewBBACA if it was not an exact match or inferred allele.
  
  - `allele_calling_evaluation`:
    - `allelecall_report.html`: An HTML report that contains the following components:
      - A table with the total number of samples, total number of loci, total number of coding sequences (CDSs) extracted from the samples, total number of CDSs classified and totals per classification type.
      - A tab panel with stacked bar charts for the classification type counts per sample and per locus.
      - A tab panel with detailed sample and locus statistics.
      - If a .tsv file with annotations is provided to the `--annotations` parameter, the report will also include a table with the provided annotations. Otherwise, it will display a warning informing that no annotations were provided.
      - A Heatmap chart representing the loci presence-absence matrix for all samples in the dataset.
      - A Heatmap chart representing the allelic distance matrix for all samples in the dataset.
      - A tree drawn with `Phylocanvas.gl` based on the Neighbor-Joining (NJ) tree computed by FastTree.
  
    - `cgMLST_MSA.fasta`: contains the MSA of the core loci. For each locus in the core genome, the alleles found in all samples are translated and aligned with MAFFT. The alignment files are concatenated to generate the full alignment. **This file can be loaded into MEGA to generate an aminoacid change matrix that can then be sent to the researchers if it's the case.**
  
    - `cgMLST_profiles.tsv`: contains the allelic profiles for the set of core loci.
  
    - `distance_matrix_symmetric.tsv`: contains the symmetric distance matrix. The distances are computed by determining the number of allelic differences from the set of core loci (shared by 100% of the samples) between each pair of samples.
  
    - `masked_profiles.tsv`: contains the masked allelic profiles (results from masking the allelic profiles in the `results_alleles.tsv` file generated by the AlleleCall module). We will use this file for the [**GrapeTree Minimun Spaning Tree** procedure that we'll see in a while.

    - `presence_absence.tsv`: Contains the loci presence-absence matrix.
  
    - `report_bundle.js`: A JavaScript bundle file necessary to visualize the report.

> [!NOTE]
> For more information, see the [**ChewBBACA's documentation**](https://chewbbaca.readthedocs.io/en/latest/index.html) in the `User Guide` section.

### `RESULTS` folder

Once you have gone through all these folders and run all the required scripts, you should go to the `RESULTS` folder and have a file called `lablog_mlst_results` (usually along with other lablog files). The `lablog` file corresponding to the chewBBACA service does the following after being run:
  * Creates a folder ending in *entrega01*.
  * Creates, inside this folder, a subfolder called `mlst`.
  * Inside this subfolder, it creates symbolic links to the following files:
    * `allelecall_report.html` from `allele_calling_evaluation`.
    * `distance_matrix_symmetric.tsv` from `allele_calling_evaluation`.
    * `results_alleles.tsv` from `allele_calling_evaluation`.
    * `cgMLST_MSA.fasta` from `allele_calling_evaluation`.
    * The `.nwk` and `.svg` files from `03-grapetree`.

## Results interpretation and service delivery

Once the whole process is finished, within the `DATE_ANALYSIS01_CHEWBBACA` folder, we should check the following files:

- `02-chewbbaca/analyze_schema/schema_report.html`:
  - In the `Schema Summary Data`:
    - Make sure that **Valid Alleles == Alleles**.
  
- `02-chewbbaca/allele_calling_evaluation/allelecall_report.html`:
  - In the `Results Summary Data`: Make sure that the number of alleles !=(EXC|INF) is not very high compared to the Total Loci.
  - In the `Counts Per Sample` plot: Look if there is a specific sample that has a lot of alleles !=(EXC|INF).
- `02-chewbbaca/allele_calling_evaluation/distance_matrix_symmetric.tsv`: Look at the distances between the samples and compare them with what you see in GrapeTree.
- `03-grapetree/tree.svg`: Check that outbreak samples are grouped together and compare it with the distance matrix.

> [!WARNING]
> While revising the results, if you see a sample that is considerably far away from the others and that is assigned to another bacteria or any other problem, remove it and run chewBBACA again. Don't hesitate to ask the team about this if you have any doubts.

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

    $ buisciii finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (no folders or files are deleted in this case). Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/bioinfo_doc/` folder.

    $ buisciii bioinfo-doc SRVCNMXXX.X

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

>[!WARNING]
>When running the `delivery` mode of the `bioinfo_doc` module, you will be asked for **delivery notes** and **email notes**. **THESE ARE NOT THE SAME THING**. After running the `service_info` mode of this module, you'll see a folder for the service will have been created in `bioinfo_doc`. There, you can for example create two files: `delivery_notes.txt` and `email_notes.txt`. Edit these two files, and add the following information in each one of them:
>* `delivery_notes.txt`: `Results were delivered in the SFTP.` (literally)
>* `email_notes.txt`: everything you want the researcher to be aware of.

>[!WARNING]
> If chewBBACA is run as part of an **outbreak** service (as mentioned before, this service is usually not done alone), you will have generated a **summary table** with all the main results from the service. When running the `delivery` mode of `bioinfo_doc`, the last question **you will be asked will be whether you want to attach any files**. **Say yes, and paste the path to the XLSX summary table you created.**

Lastly, once the service has been delivered and the e-mail has been sent, remember to remove all the files related to this service from `scratch_tmp`:

    $ buisciii scratch SRVCNMXXX.X
    $ remove_scratch

## Outbreak report template

This is just a general template, but feel free to do all the adjustments you need.

```
En este servicio se ha realizado ensamblado de novo, caracterización de resistencias, análisis de factores de virulencia, análisis de plásmidos, análisis de SNPs, análisis MLST y análisis cgMLST.

Adjuntamos tabla resumen (summary_outbreak_XXXX.xlsx) de los resultados de los análisis actualizados, hemos realizado:
- Summary:
    - Análisis MLST.
    - Análisis de identificación de especie y contaminaciones con Kmerfinder.
    - Estadísticas de mapeo de las muestras frente a la referencia XXXX.
    - Ensamblado y anotación de novo.
- Análisis filogenético con matriz de SNPs. Aquí hay dos resultados:
   - Sin filtrar zonas recombinantes (snpmatrix_all_pos y snpmatrix_all_pos_pairs).
   - Filtrando zonas recombinantes (snpmatrix_core y snpmatrix_core_pairs).
- Análisis de plásmidos.
- Análisis de factores de virulencia.
- Búsqueda de genes de resistencia con las bases de datos CARD y AMRFinderPlus.
- Análisis MLVA.

Además, en la carpeta RESULTS, encontrarás todos los resultados que están presentes en la tabla: 
├── assembly
│   ├── assemblies: ficheros fasta de los ensamblados.
│   ├── kmerfinder_summary.csv: resultados de kmerfinder para las muestras.
│   ├── multiqc_report.html: report en html de la calidad del ensamblado y la anotación para las muestras.
│   ├── quast_XXXX_report.html: resultado de quast con la calidad del ensamblado frente a dicha referencia.
│   ├── quast_global_report.html: resultado global de quast con la calidad del ensamblado.
│   └── summary_assembly_metrics_mqc.csv: tabla resumen excel con resultados de la calidad del ensamblado.
├── characterization
│   ├── amrfinderplus: resultados de resistencia a antibióticos con AMRFinderPlus.
│   ├── ariba_card.csv: resultados de resistencia a antibióticos con ariba y base de datos CARD.
│   ├── ariba_plasmidfinder.csv: resultados de identificación de plásmidos y base de datos PlasmidFinder.
│   ├── ariba_vfdb_full.csv: resultados de genes de virulencia con ariba y base de datos VFDB.
│   ├── ariba_mlst_full.tsv: resultado MLST.
├── snp
|   ├── mapping_stats_summary.txt: estadísticas de mapping.
|   ├── phylo.iqtree.bootstrap.nwk: árbol en formato newick, puede ser usado para ver el árbol en MEGA, Figtree, iTOL...
|   ├── clean.core.iqtree.bootstrap.nwk: árbol en formato newick, habiendo filtrado zonas recombinantes, puede ser usado para ver el árbol en MEGA, Figtree, iTOL...
|   ├── variants_stats.txt: estadísticas de número de variantes.
|   |── wgs_metrics_all_filtered.txt: estadísticas más extensas de mapping.
├── plasmidid:
|   ├── <sample_name>_<plasmid_id>.png: Figura de reconstrucción del plásmido para una muestra dada. Se incluye anotación de resistencias.
│   ├── <sample_name>_summary.png: Imagen con el gráfico resumen de todos los plásmidos identificados para una muestra dada.
|   ├── <sample_name>_final_results.html: Fichero HTML con los resultados de PlasmidID para una muestra dada.
├── mlva:
│   ├── MLVA_analysis_assemblies.csv: tabla que contiene los valores MLVA para cada ensamblado en todos los loci del análisis.
│   ├── assemblies_mismatchs.txt: diferencias para cada locus (solo loci con diferencias).
│   ├── assemblies_output.csv: tabla que contiene toda la información del análisis, posición de los primers en cada match, tamaño del insert, número de diferencias (mismatches), etc.
│   ├── predicted-pcr-size-table-assemblies.csv: tabla que contiene los tamaños de PCR predichos.
├── mlst
│   ├── allelecall_report.html: informe en formato HTML con estadísticas de la llamada de alelos habiendo empleado el esquema generado a partir de los genomas enviados por correo.
|   ├── cgMLST_MSA.fasta: MSA de los loci del core.
|   |── results_alleles.tsv: perfiles alélicos determinados para las muestras y los genomas adicionales.
|   |── distance_matrix_symmetric.tsv: matriz simétrica de distancias para cada par de muestras.
|   |── grapetree: árbol de expansión mínima en formato .svg y .nwk, puede ser usado para ver el árbol en MEGA, Figtree, iTOL...
└── summary_outbreak_XXXX.xlsx: Tabla resumen adjunta en el correo.
```
---
