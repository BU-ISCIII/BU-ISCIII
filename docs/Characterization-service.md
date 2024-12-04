# How to perform the Characterization service

Welcome to this tutorial on how to perform a **Characterization Service** as a member of the ISCIII's Bioinformatics Unit!

**First of all**, when using the "Characterization" term, we refer to the **Bacteria: Multi-Locus Sequence Typing (MLST), analysis of virulence factors, antimicrobial resistance, and plasmids characterization** service that researchers can select when requesting a service. Usually, this service is requested along with the **Bacteria: *de novo* genome assembly and annotation**, **Bacteria: Plasmid analysis and characterization** and **Fungal / bacteria / virus : Variant calling, annotation and SNP-based outbreak analysis (e.g. haploid fungal outbreak)** services, which correspond to the **assembly**, **plasmidid** and **snippy** templates.

## Introduction

### Service request

Within which context is this service carried out? In general, the researcher that requests this service provides an external file (usually an Excel file) indicating the samples that must be analysed, which species they correspond to, etc., unless there aren't a lot of samples or they are all related to the same species.

Considering this information, they will ask for the **assembly** of the samples, followed by the **identification** of **virulence factors**, **antibiotic resistances** and **plasmids**, and an **SNP analysis**:
* For the **assembly** of the samples, the bioinformatics procedure that must be carried out is explained in detail the **[Assembly service](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) guide**.
* For the **identification of virulence factors, resistances and plasmids**, the procedure to follow will be explained in **this guide**.
* For the identification and characterization of **plasmids**, this procedure is also done bioinformatically by means of **PlasmidID**. You can find a guide on how to use it **[here](https://github.com/BU-ISCIII/BU-ISCIII/wiki/plasmidID-service)**.
* For the **SNP analysis** of the samples, this is done bioinformatically with the **[SNIPPY](https://github.com/tseemann/snippy)** software. You can find all the necessary information on how to use it in the **[SNP analysis](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service)** guide.

### Software used

For this Characterization service (specifically, for the identification of virulence factors, antibiotic resistances and plasmids), the main tool that is used is **[ARIBA](https://github.com/sanger-pathogens/ariba)**. This software identifies antimicrobial resistance (AMR) genes from paired sequencing reads, employing databases such as **[CARD](https://card.mcmaster.ca/)** or **[ResFinder](http://genepi.food.dtu.dk/resfinder)**. It can also be used for **multi-locus sequence typing** (**MLST**) and for the identification of **virulence factors** and **plasmids**, by relying on the **[PubMLST](https://pubmlst.org/)**, **[VFDB](http://www.mgc.ac.cn/VFs/main.htm)** and **[PlasmidFinder](https://cge.food.dtu.dk/services/PlasmidFinder/)** databases, respectively.

Apart from ARIBA, two other tools are used within this service:
* **[emmtyper](https://github.com/MDU-PHL/emmtyper)**: this is a tool that is employed for the characterization of *emm*-coding genes in *S. pyogenes*. This is relevant since *S. pyogenes* can be classified into more than 275 *emm* types, and typing is based on sequence analysis of part of the M protein gene (*emm*), with high importance in epidemiological studies ([**source**](https://www.cdc.gov/strep-lab/php/group-a-strep/emm-typing.html)).
* **MLVA**: by means of the `MLVA_finder.py` file (obtained from the [**MLVA_finder GitHub page**](https://github.com/i2bc/MLVA_finder)), this script is used for **multi-locus VNTR analysis**, a molecular typing method to subtype microbial isolates based on the Variable copy Numbers of Tandem Repeats (VNTR).

## Bioinformatics procedure

Taking the previous information into account, the first thing we need to do is take the service and click on _**Add resolution**_ in **[iskyLIMS](https://iskylims.isciii.es/)** after loggin in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym.

In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:

```shell
cd /data/bi/services_and_colaborations/CNM/bacteriology
ll -tr
```

Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered.

At this point, you'll have to consider what kind of service has been requested in this case. In general, follow these guidelines:
* If all samples correspond to a specific bacterial species, within the context of an outbreak, the service acronym should have the following structure: `SPECIESOUTBREAKXXX`. For example, if the outbreak corresponds to *Bacillus cereus* and this is the first outbreak we analyse in relation to this species, the service should be called `BCEREUSOUTBREAK001`.
* If the service is related to one or several species, but it is not within the context of an outbreak, the service should be called after the following structure: `WGSORGANISMXX`. For example, if we have samples from several species from the same genus (like *Burkholderia*), we should call the service `WGSBURKHOLDERIA01` (if it's the first service related to *Burkholderia* species).
* If the service corresponds to samples from several species, not necessarily from the same genus, and therefore not all species can be categorised inside one name, we should call the service after the following structure: `CHARACTERIZATIONXX`. For example, if our service includes samples with a wide variety of species from different genera, we should call the service `CHARACTERIZATION01` (if it's the first service related to multiple species).

Regardless of the acronym that has been assigned to the service, your service acronym will be the last service's number + 1. For example, if you have chosen the `CHARACTERIZATION` acronym for your service and the last folder after executing `ll -tr` is `SRVCNM1200_20240823_CHARACTERIZATION07_svaldezate_S`, the service acronym for your new service will be _**CHARACTERIZATION08**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools micromamba environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder:

```shell
cd /data/bi/services_and_colaborations/CNM/bacteriology/
ll -tr
```

Now, let's execute the first BU-ISCIII tool: `new-service`, where you'll need to specify the resolution ID associated to this service.

```shell
bu-isciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X
```

The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results).  

Once `new-service` is executed, you'll be asked:

* `Do you want to skip folder creation?`: Unless it is not the first resolution associated with the service, answer **NO**, because the folder corresponding to the service has not yet been created in the `services_and_collaborations` folder.

* Next, specify `characterization`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can then get into the `ANALYSIS` folder and we'll find the following lablog file inside:

* `lablog_characterization`: an executable file that creates the `00-reads` folder which will contain all the raw reads, apart from renaming the `ANALYSIS01_CHARACTERIZATION` folder so that it contains the analysis date. Please remember to change the number associated to the word `ANALYSIS` accordingly, depending on whether other services have been done previously or not.

Let's execute the `lablog_characterization` file:

```shell
bash lablog_characterization
```

Once this file has been executed, please take into consideration that this service is usually performed along with other pipelines (normally **assembly**, **plasmidid** and **snippy**), so run all the necessary lablog files before moving on to the next BU-ISCIII module.

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/bi`, since this `scratch_tmp` folder will be the one used for the analysis.

```shell
bu-isciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
```

Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S/ANALYSIS/DATE_ANALYSIS02_CHARACTERIZATION
```

> [!WARNING]
> Please note that the Characterization service is usually performed along with the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service); that's why this folder is called `ANALYSIS02`. During this assembly service, **we should have saved the trimmed sequences**, since **they will be needed** during the Characterization analysis.

Once we're at this point, and before executing anything else, we should load all the necessary dependencies:

```shell
module load Nextflow singularity
```

If everything is OK, we can then get into the `ANALYSIS` folder and we'll find the following items inside:

* `lablog_characterization`, which was already executed.
  
* `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

* `ANALYSIS01_CHARACTERIZATION`: a folder containg several subfolders whose scripts will be launched **sequentially** from now on:

  * `lablog`: this file creates symlinks to the `00-reads` folder and the `samples_id.txt` file located in the `ANALYSIS02_CHARACTERIZATION` folder.

  * `01-preprocessing`: this folder contains a `lablog` file that creates a subfolder for each sample, inside which there will be symlinks to the fastp processed and compressed fastq files for the corresponding samples linked to the service. The user will be asked whether these trimmed reads were saved previously (since this Characterization service is usually performed along with the Assembly service, **these trimmed reads should have already been saved when performing this procedure**). If the user says yes, these symbolic links are created; otherwise, a `_01_fastp.sh` script will be created, which will have to be launched in order to obtain the trimmed sequences from the raw reads, which will be stored within this `01-preprocessing` folder.
  
  * `02-ariba`: this folder contains the following elements:
  
    * `lablog`: this file copies a file called `databases.txt` from `/data/bi/references/` to the current directory.
  
    * `databases.txt`: this file indicates the databases that will be used when launching the ariba software. By default, these databases are **card**, **plasmidfinder** and **vfdb_full** but, if you see your organism/s are included in the [**PubMLST**](https://pubmlst.org/) database, you should manually modify this .txt file in order to include **pubmlst**. Once this .txt file has been checked, proceed with the `run` subfolder:
  
    * `run`: a folder which contains:
  
      * `lablog`: this file does several things:
        * First, it checks whether the reference folder that will be used by ARIBA in relation to the PubMLST database is stored within REFERENCES.
          > [!WARNING]
          > Before running this lablog file, we should:
          > 1. Check whether the species being analysed for this service have an MLST schema in the PubMLST database.
          > 2. If so, we should download this schema and store it in `REFERENCES`. How? In `REFERENCES`, you should find a script called `lablog`. If you launch this script, you will be asked to select a genome from PubMLST, given a list of available genomes. Write the number associated to your genome of interest, and press Enter.
          > 3. After that, you'll find a script called `_01_download_pubmlst.sh`. Run this script, and you'll then find a folder, named after today's date, in which there'll be a subfolder called `ref_db`. This is the folder that will be used by ARIBA when running `run/lablog`.
        * Then, it creates a file called `sample_database.txt`, which will have two columns: one for the sample name and the other one for the corresponding database being used by ARIBA. Each sample name will therefore appear as many times as databases are written in `databases.txt`.
        * Then, it creates the following scripts, that should be run sequentially:
          * `_01_ariba.sh`: this script will run ARIBA for all the databases indicated previously in `databases.txt`, using the trimmed reads stored in `01-preprocessing` as input.
  
          * `_02_fix_tsvreport.sh`: this script renames the .tsv reports from ARIBA so that they contain the sample name and the corresponding database.
  
      After running this `lablog` file and the other scripts, inside this `run` folder, we'll find a subfolder for each sample, inside of which we'll find, at the same time, as many subfolders as databases being employed by ARIBA. Inside these subfolders, we'll find several files, including the .tsv reports that were mentioned before.
  
    * `summary`: a folder which contains:
  
      * `lablog`: this file creates the `_01_ariba_summary_prueba.sh` script, that uses the [ariba summary](https://github.com/sanger-pathogens/ariba/wiki/Task:-summary) function in order to summarise the results from ARIBA for all the databases that were employed.
  
      In the end, we'll have, for each database, three files:
      * `out_summary_XXXX.csv`: a .csv summary file.

      * `out_summary_XXXX.phandango.csv` and `out_summary_XXXX.phandango.tre`: these two files can be loaded into [Phandango](https://jameshadfield.github.io/phandango/#/) in order to visualise the results, even though the visualisation is very general: the graph is simply saying whether or not each sample has a 'match', for example for a resistance gene from CARD or a housekeeping gene from the MLST schema. Green means a match, and pink means not a match.
  
  * `03-amrfinderplus`: this folder contains a `lablog` file that:
    * Creates a `logs` folder.
  
    * Given a list of organisms for which resistances can be specifically determined, this list is shown to the user, so that they can write the number corresponding to the organims of interest. If such organism does not appear on the list, the user can simply write **26**, which corresponds to another organism; AMRFinder will still work.
  
    * Creates a `_01_run_amrfinder.sh` script, which should be run next.

    >[!WARNING]
    > Please remember to **activate** the appropriate **micromamba** **environment** before running `_01_run_amrfinder.sh`:
    >```
    >micromamba activate amrfinder_3.12.8
    >```

    After running this script, a .tsv report from AMRFinder will be obtained within this folder for each sample. To briefly summarize, the "Method" column combined with the % identity indicates how a hit was identified, and whether the protein is a partial, an exact match to a known sequence, etc. This should give an indication of how confident you should be that this is a full-length functional gene. The "core" vs. "plus" column should help you to determine the level of curation that went into the inclusion of that gene in the database. The Sequence name, Type, Subtype, Class, and Subclass provides an estimation of the category and function for the gene. You can find more information on the interpretation of results [**here**](https://github.com/ncbi/amr/wiki/Interpreting-results).

  * `04-emmtyper`: this folder corresponds to *emm*-typing of *Streptococcus pyogenes* by means of the U.S. Center for Disease Control and Prevention (CDC) trimmed *emm* subtype database. **Ignore this folder if *emm*-typing is not requested**. This folder contains a `lablog` file that:
    * Creates two folders called `fasta_inputs` and `logs`.

    * Creates a file called `assembly_file_list.txt` inside `fasta_inputs`, which will contain paths to the assemblies obtained previously from Unicycler during the assembly procedure.
  
    * Creates a script called `_00_unzip_jobarray.sbatch` which will decompress the assemblies.
  
    * Creates a script called `_01_emmtyper.sbatch` which will run emmtyper with the blast mode, in which contigs are BLASTed against the trimmed FASTA database curated by the CDC.
  
    * Creates a final script called `_ALLSTEPS_emmtyper.sh`, which chains both scripts so that they are run sequentially.
  
    The output is stored within a subfolder called `01-typing`, in a file called `results_emmtyper.out`. This tab-separated file will contain information such as the number of BLAST hits, the predicted *emm*-type, possible *emm*-like alleles, etc.
  
  * `05-mlva`: this folder corresponds to MLVA. **Ignore this folder if the researcher has not indicated they need MLVA to be performed in their petition**. This folder contains a `lablog` file that does the following:
    * Creates the subfolders `logs`, `assemblies` and `MLVA_output`.
  
    * Copies the assemblies that were obtained from Unicyler during the previous assembly procedure into the `assemblies` folder, and decompresses them.
  
    * Inside `/data/bi/references/MLVA/`, there are some .txt files, corresponding to a list of primers to recover sequences from the VNTRs, that will be used by the `MLVA_finder.py` script later. This `lablog` file will ask the user which primer file to use, depending on the organism being analysed. Once the user selects the file, a `_01_mlva.sh` script is created, which should be executed next.
  
    Once `_01_mlva.sh` has been launched, using the assemblies as input and the primer file that we indicated previously, we'll find the following files within the `MLVA_output` subfolder:
    * `MLVA_analysis_assemblies.csv`: a .csv file containing all MLVA values for each assembly and all loci from the MLVA analysis. This .csv file is designed to be uploaded on http://microbesgenotyping.i2bc.paris-saclay.fr.
    * `assemblies_output.csv`: a .csv file containing all the information from the MLVA analysis such as primers positions of match(s), size of insert, number of mismatches, etc.
    * `assemblies_mismatchs.txt`: a .txt file containing all different mismatches for each locus (only loci with mismatches) found during the MLVA analysis on input sequences.
    * `predicted_PCR_size_table_assemblies.csv`: a .csv file containing predicted PCR size.

  * `99-stats`: this folder contains a `lablog` file that will use a script called `parse_ariba.py` in order to create reports with all the relevant information for each database that was employed:
    * For **CARD**: the .csv file will show any matches with resistance genes for each sample.
    * For **PubMLST**: the .tsv report will show the Strain Type (ST) each sample belongs to, along with their corresponding MLST profile.
    * For **PlasmidFinder**: the .csv file shows all plasmids found for each sample.
    * For **VFDB**: the .csv file shows the number of virulence genes found for each sample, along with their names.

Once you have gone through all these folders and run all the required scripts, you should go to the `RESULTS` folder and have a file called `lablog_characterization_results` (usually along with `lablog_assembly_results`, `lablog_plasmidid_results` and `lablog_snippy_results`). The lablog corresponding to the Characterization service does the following:
  * Creates a folder ending in *entrega01*.
  
  * Creates, inside this folder, a subfolder called `characterization`, which will have two subfolders, corresponding to `amrfinderplus` and `emmtyper`.
  
  * Creates symbolic links to the .tsv and .csv reports from `99-stats`.
  
  * Inside `amrfinderplus`, creates symbolic links to all .tsv reports from AMRFinderPlus.
  
  * Inside `emmtyper`, creates a symbolic link to the `results_emmtyper.out` file obtained after running emmtyper.

## What should I do after completing the service?

Once we are done with the service (including the assembly, plasmidid and snippy procedures, since this is the usual case), we'll have to review the results from each procedure and add all the relevant information into an Excel file. For this, we use a **template** that you can find [**here**](https://docs.google.com/spreadsheets/d/1m_hnCGNgtWcoJAjs_91BkmfJn6CNr4yO/edit?usp=drive_link&ouid=108428245306738036878&rtpof=true&sd=true).

In this Excel file, we can find the following sheets:
* **summary**: this sheet contains information regarding the assembly performed previously (check the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) page), Kmerfinder, the MLST profile identified for the samples and the mapping procedure done against a reference with snippy (check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page).
  * The **MLST profile** can be extracted from the `99-stats/ariba_mlst_full.tsv` file.
  * The **Kmerfinder** columns `07-kmerfinder_best_hit_# Assembly`, `07-kmerfinder_best_hit_Accession Number` and `07-kmerfinder_best_hit_Description` can be extracted from `*ASSEMBLY/99-stats/kmerfinder_summary.csv`.
  * The **MAPPING** columns are relative to snippy. The way to fill in these columns is explained on the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page.
  * The **ASSEMBLY** columns are relative to the assembly service:
    * The columns `# contigs (>= 1000 bp)`, `GC (%)`, `L50`, `N50` and `Total length (>= 1000 bp)` can be extracted from the `*ASSEMBLY/03-assembly/quast/global_report/transposed_report.tsv` file.
    * The columns `# genomic features` and `Genome fraction (%)` can be extracted from the `*ASSEMBLY/03-assembly/quast/per_reference_reports/XXXX/transposed_report.tsv` file.
* **snpmatrix_all_pos**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page.
* **snpmatrix_all_pos_pairs**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page.
* **snpmatrix_core**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page.
* **snpmatrix_core_pairs**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page.
* **plasmids**: this sheet is filled in using the results from **ARIBA** with the **PlasmidFinder** database along with the ones from **PlasmidID**. In this sheet, we can find 8 columns: `sample`, `plasmidID`, `length`, `species`, `description`, `fraction_covered`, `contig_number` and `%mapping`. All these fields can be completed by checking the **PlasmidID** reports (check the [**PlasmidID service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/plasmidID-service) page).
  * In the case of **PlasmidFinder**, check the `99-stats/ariba_plasmidfinder.csv` and `02-ariba/summary/out_summary_plasmidfinder.csv` files. You'll be able to see any possible matches of the samples with plasmids stored in PlasmidFinder, along with their IDs, which you can then search on [**NCBI**](https://www.ncbi.nlm.nih.gov/). You'll therefore be able to complete at least the columns called `sample`, `plasmidID`, `length`, `species` and `description`.
* **virulence**: there are three columns in this sheet: `sample`, `Virulences VFDB` and `Number of virulence genes`. This information can be extracted from the `99-stats/ariba_vfdb_full.csv` file.
* **Resistance result**: there are three columsn in this sheet: `sample`, `CARD` and `AMRFinderPlus`. In these two last columns, you'll simply have to list the identified resistance genes according to both databases, separated by commas.
  * The **CARD**-related information can be obtained from `99-stats/ariba_card.csv`.
  * The **AMRFinderPlus**-related information can be extracted from each .tsv report from the `03-amrfinderplus` folder. Specifically, you need to list all genes from the column `Gene symbol` from these reports.
* **AMRFinderPlus Resistance result**: in this sheet, you'll find all columns from the .tsv reports obtained from amrfinder, except for the columns `Protein identifier`, `HMM id` and `HMM description`. You'll have to open each .tsv report, copy and paste these columns into this sheet.
* **MLVA**: in this sheet, which initially is blank, you should have a first column corresponding to the sample names, and then as many columns as loci were analysed during the procedure. In order to fill in this sheet, copy and paste the content from `05-mlva/MLVA_output/MLVA_analysis_assemblies.csv`.

Fill in the Excel template following the previous instructions, name it according to the structure `summary_outbreak_species.xlsx`, and then do the following:
* Being logged in your Google account, go to the **buisciii_shared** Drive folder.
* Go to **ALERTS**.
* Create a new subfolder called after the organism of interest followed by the date in which the service was requested. For example, if the service that we are carrying out is relative to *Brucella melitensis* and the service was requested on 28/10/2024, the new folder will have this name: `BMELITENSIS_20241028`.
* Go inside this subfolder and store the Excel report in here.

Once the Excel file has been completed with all the necessary information and has been updated into the corresponding Drive folder, **save a copy of this Excel file** both inside the `RESULTS/DATE_entrega01` folder and the `ANALYSIS` folder of the service. You can then proceed with the copy of the service to `/data/bi/services_and_colaborations/CNM/bacteriology/` and `/data/bi/sftp`.

---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools.

    $ bu-isciii --log-file SRVCNMXXX.X.finish.log finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (in **Characterization**, this folder is `01-preprocessing`). Then, it copies all the service files back to its `/data/bi/services_and_colaborations/CNM/bacteriology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/bioinfo_doc/` folder.

    $ bu-isciii --log-file SRVCNMXXX.X.tool.log bioinfo-doc SRVCNMXXX.X

Remember to save the logs with the corresponding name (i.e. `SRVCNMXXX.X.service_info.log` or `SRVCNMXXX.X.delivery.log`).

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

>[!WARNING]
> When performing an **outbreak** service (as mentioned before, this Characterization service is usually not done alone, but along with the assembly, plasmidid and snippy procedures), the delivery message is in general too long to be included as a .txt file during the delivery procedure. Therefore, for this kind of services, reply the following when these questions are asked on the terminal:
>1. Do you wish to provide a text file for delivery notes?: type n.
>2. Write some delivery notes: leave it blank, by pressing Enter.
>3. Do you want to add some delivery notes to the e-mail?: type n.
>4. Do you want to send e-mail automatically?: type n.
>
>The service_info and delivery .pdf files will have been created but the e-mail won't have been sent. This will be done manually by sending the service report that you can see in the last section of this manual.

Lastly, once the service has been delivered and the e-mail has been sent, remember to remove all the files related to this service from `scratch_tmp`:

    $ bu-isciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
    $ remove_scratch

## Outbreak report template

This is just a general template, but feel free to do all the adjustments you need.

```
En este servicio se ha realizado ensamblado de novo, caracterización de resistencias, análisis de factores de virulencia, análisis de plásmidos, análisis de SNPs y análisis MLST.

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
└── summary_outbreak_XXXX.xlsx: Tabla resumen adjunta en el correo.
```
---
