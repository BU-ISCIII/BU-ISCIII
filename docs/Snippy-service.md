# How to perform the Snippy service

Welcome to this tutorial on how to perform a **Snippy Service** as a member of the ISCIII's Bioinformatics Unit!

**First of all**, when using the "Snippy" term, we refer to the **Fungal / bacteria / virus : Variant calling, annotation and SNP-based outbreak analysis (e.g. haploid fungal outbreak)** service that researchers can select when requesting a service. Usually, this service is requested along with the **Bacteria: *de novo* genome assembly and annotation**, **Bacteria: Plasmid analysis and characterization** and **Bacteria: Multi-Locus Sequence Typing (MLST), analysis of virulence factors, antimicrobial resistance, and plasmids characterization** services, which correspond to the **assembly**, **plasmidid** and **characterization** templates, respectively.

## Introduction

### Service request

Within which context is this service carried out? In general, the researcher that requests this service provides an external file (usually an Excel file) indicating the samples that must be analysed, which species they correspond to, etc., unless there aren't a lot of samples or they are all related to the same species.

Considering this information, they will ask for the **assembly** of the samples, followed by the **identification** of **virulence factors**, **antibiotic resistances** and **plasmids**, and an **SNP analysis**:
* For the **assembly** of the samples, the bioinformatics procedure that must be carried out is explained in detail the **[Assembly service](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) guide**.
* For the **identification of virulence factors, resistances and plasmids**, please follow the instructions from the [**Characterization guide**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* For the identification and characterization of **plasmids**, this procedure is also done bioinformatically by means of **PlasmidID**. You can find a guide on how to use it **[here](https://github.com/BU-ISCIII/BU-ISCIII/wiki/plasmidID-service)**.
* For the **SNP analysis** of the samples, this is done bioinformatically with the **[SNIPPY](https://github.com/tseemann/snippy)** software. Its usage will be explained **in this guide**.

### Software used

For this service, two main programmes are employed in order to perform **SNP analysis** and **create a phylogenetic tree** for the samples being analysed:
* [**SNIPPY**](https://github.com/tseemann/snippy): Snippy aligns the previously processed reads against a reference genome and performs **variant calling** using [Freebayes](https://github.com/ekg/freebayes), finding:
  * **Single Nucleotide Polymorphisms** (SNPs)
  * **Multiple Nucleotide Polymorphisms** (MNPs)
  *  **Insertions**
  *  **Deletions**
  *  Combinations of SNPs and MNPs (**complex variants**).
  
  Then, once variants have been identified for each sample, an SNP alignment file, including both variant and invariant sites, is generated.
  
  Snippy then uses [**SNP-sites**](https://www.microbiologyresearch.org/content/journal/mgen/10.1099/mgen.0.000056) to extract SNPs from this multi-FASTA alignment, generating another file containing the SNP sites that can be used for the generation of an **SNP matrix** with the **pairwise distances** among the samples, including transitions and transversions and establishing uniform rates among sites.

  During this procedure, the [**Gubbins**](https://github.com/nickjcroucher/gubbins/blob/master/docs/gubbins_manual.md) software is employed. This algorithm iteratively identifies loci containing elevated densities of base substitutions, which are considered **recombinations**. Gubbins is used as part of this service in order to exclude recombinant sites from the multi-FASTA alignments and from the SNP matrices.
  
  **This whole procedure will be shown step-by-step through this guide.**
* [**IQTREE**](http://www.iqtree.org/): Iqtree is a phylogenetic tree generation programme that estimates the tree that most adequately represents the evolutionary relationships between the evaluated strains, by selecting the best [**substitution model**](http://www.iqtree.org/doc/Substitution-Models).

* [**MEGA**](https://www.megasoftware.net/): this software can be used for multiple purposes within the context of **evolutionary analysis**, including the selection of the best-fit substitution model, the estimation of evolutionary distances and divergence times, the reconstruction of phylogenies, the prediction of ancestral sequences and the diagnosis of disease mutations ([Caspermeyer, 2018](https://doi.org/10.1093/molbev/msy098)). In our case, we'll use it to obtain the **SNP matrix** with the pairwise distances among the samples that was mentioned previously.

## Bioinformatics procedure

>[!WARNING]
>As stated before, the SNP-analysis service is usually not done alone, but along with de novo assembly, characterization and PlasmidID. Therefore, you'll most likely already have created the service folder, which will still have the acronym associated to the Characterization service, as indicated below.

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
buisciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X
```

The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results).  

Once `new-service` is executed, you'll be asked:

* `Do you want to skip folder creation?`:
  * **If the service folder has not been created yet in the `services_and_collaborations` folder**, answer **NO**.
  * **If this is not the first resolution associated with the service or another service has already been performed for the current resolution**, answer **YES**.

* Next, specify `snippy`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can then get into the `ANALYSIS` folder and we'll find, apart from folders and files from other services done previously, the following `lablog` file inside:

* `lablog_snippy`: an executable file that creates the `00-reads` folder which will contain all the raw reads, apart from renaming the `ANALYSIS01_SNIPPY` folder so that it contains the analysis date. Please remember to change the number associated to the word `ANALYSIS` accordingly, depending on whether other services have been done previously or not.

Let's execute the `lablog_snippy` file:

```shell
bash lablog_snippy
```

Once this file has been executed, please take into consideration that this service is usually performed along with other pipelines (normally **assembly**, **plasmidid** and **snippy**), so **run all the necessary `lablog` files before moving on to the next BU-ISCIII module**.

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the analysis. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

```shell
buisciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
```

Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S/ANALYSIS/DATE_ANALYSIS04_SNIPPY
```

> [!WARNING]
> Please note that the Snippy service is usually performed along with the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service), the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) and the [**PlasmidID service**](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/plasmidID-service.md); that's why this folder is called `ANALYSIS04`. During the assembly service, **we should have saved the trimmed sequences**, since **they will be needed** during the Snippy procedure.

Once we're at this point, and before executing anything else, we should load all the necessary dependencies:

```shell
module load singularity
```

If everything is OK, we can then get into the `ANALYSIS` folder and we'll find the following items inside:

* `lablog_snippy`, which was already executed.
  
* `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.

* `ANALYSIS01_SNIPPY`: a folder containg several subfolders whose scripts will be launched **sequentially** from now on:
  * `lablog`: this file creates symlinks to the `00-reads` folder and the `samples_id.txt` file located in the `ANALYSIS02_CHARACTERIZATION` folder.

  * `01-fastqc`: this folder contains a `lablog` file that runs FASTQC on the raw reads of the samples. Usually, this `lablog` is not run, since this procedure has already been done as part of the Assembly service.
  * `02-preprocessing`: this folder contains a `lablog` file that creates a subfolder for each sample, inside of which there will be symlinks to the fastp processed and compressed .fastq files for the corresponding samples linked to the service. The user will be asked whether these trimmed reads were saved previously (since this Snippy service is usually performed along with the Assembly service, **these trimmed reads should have already been saved when performing this procedure**). If the user says yes, these symbolic links are created; otherwise, a `_01_fastp.sh` script will be created, which will have to be launched in order to obtain the trimmed sequences from the raw reads, which will be stored within this `02-preprocessing` folder.
  * `03-preprocQC`: this folder contains a `lablog` file that runs FASTQC on the processed reads obtained from fastp. Usually, this lablog is not run, since this procedure has already been done as part of the Assembly service.
  * `04-snippy`: this folder contains a `lablog` file that does the following:
    * Creates a subfolder called `logs`.
  
    * Creates a file called `input.tab` that, for each sample, lists the .fastq files that are linked to that sample, separated by tabs (there will be, therefore, three columns for each sample). These .fastq files are fetched from the `02-preprocessing` folder.
        >[!WARNING] 
         >- **In some cases, you might want to include other genomes in your analysis, and these genomes will consist in .FASTA files.**
         >- **Snippy can also work on .FASTA files. In these cases, you'll only have two columns: the genome name and the associated .FASTA file.**
         >- **An `input.tab` file can have both samples with .fastq files and genomes with a .FASTA file at the same time.**

    * Creates a file called `commands.out` that will contain the output after running the function `snippy-multi` from the snippy singularity image, using the file `input.tab` as input and **a reference that must be indicated manually. This reference should be stored in the `REFERENCES` folder and must have all plasmid sequences removed before carrying on**.
  
      In order to **remove plasmid sequences** from the reference you downloaded (probably from **NCBI**), run this command:

      ```
      for file in ./*.fna; do awk '/^>/ {in_plasmid = /plasmid/} !in_plasmid' "$file" > "clean_$(basename "$file")"; done
      ```
    
      The `snippy-multi` function is used to run `snippy` on a set of isolate sequences against the same reference, apart from running the `snippy-core` function to generate the corresponding core genome SNP alignment files. All the commands, ready to be launched, will be stored in this file. The minimum number of reads covering a site to be considered as a variant (`--mincov`) is set to 9, the minimum mapping quality to accept in variant calling (`--mapqual`) is set to 10, the minimum quality a nucleotide needs to be used in variant calling (`--basequal`) is set to 5 and the minimum VCF variant call "quality" (`--minqual`) is set to 30.
    * Creates a script called `_00_snippy.sh` that runs all commands from `commands.out`, **except for the line corresponding to `snippy-core`**. This script will have as many lines as samples/genomes being analysed.
      * When executing this script, `snippy` will be launched for all samples/genomes at the same time, each one having a different slurm job ID.
      * In the end, there will be a subfolder for each sample/genome.
      * Inside each folder, you'll find different files corresponding to the identified variants, for example:
        * a **.tab** file with a summary of the variants.
        * an **HTML** version of this .tab file.
        * the variants in **.bed** format.
        * the alignments in **.bam** format (including unmapped and multimapping reads).
        * the annotated variants in **.vcf** format.
        * a compressed .vcf file.
        * etc.
    * Creates a script called `_01_snippy_core.sh`. When executing this script, it will run the last line from `commands.out` in order to obtain different files associated to the core genome, starting with `core.*`.
  
        >The objective is obtaining an **alignment** of **core SNPs**, which could be used for **phylogeny reconstruction**. A **core position** can be defined as a **genomic position that is present in all samples**, and it can be **monomorphic** (same nucleotide in all samples) or **polymorphic** (different nucleotides among samples).

        To do so, `snippy-core` takes all the subfolders that were created in the previous step as input, as well as the reference file. This function merges all .vcf files into a `core.vcf` file, containing all core variants (SNPs and indels that are common to all samples/genomes). Depending on your situation, you might want to remove certain subfolders from the command before running `_01_snippy_core.sh`.
        
        This script generates the following files:
      * `core.aln`: an **SNP alignment** of the **core genome** **only** for the **variant sites**.
      * `core.full.aln`: an **SNP alignment** of the **whole genome**, including **both variant and invariant sites**.
      * `core.ref.fa`: **reference** in .FASTA format.
      * `core.tab`: tab-separated columnar list of **core SNP sites** with alleles but **NO annotations**.
      * `core.txt`: tab-separated columnar list of **alignment/core-size statistics**.
      * `core.vcf`: multi-sample VCF file with genotype GT **tags** for **all discovered alleles**.
    * Creates a script called `_02_phylo_aln.sh`. When running this script, it launches the `snp-sites` function, using the `core.full.aln` alignment file as input and generating an output file called `phylo.aln`. **SNP-sites** is used to extract SNPs from a multi-FASTA alignment (`core.full.aln`) and, in this case, it is launched so that it outputs only columns containing exclusively ACGT (`-c` option) and monomorphic sites (`-b` option).
  
        The `phylo.aln` file consists in an **SNP alignment** of the **core genome**, including **both variant and invariant sites**.
    
    * Creates a script called `_03_gubbins.sh` that, when being launched, does the following:

      * It runs the function `snippy-clean_full_aln` from snippy using `core.full.aln` as input, replacing all unusual characters in the alignment with an `N` character. It outputs a file called `clean.full.aln`, which is the SNP alignment of the whole genome for both variant and invariant sites, but this time with no unusual characters.
  
      * It then runs the `run_gubbins.py` script from the gubbins singularity image, using `clean.full.aln` as input and "gubbins" as prefix (`-p` option) for all output files. After running this script, several files are obtained:
        * `gubbins.recombination_predictions.embl`: recombination	predictions	in EMBL file format.
        * `gubbins.recombination_predictions.gff`: recombination predictions in GFF format.
        * `gubbins.branch_base_reconstruction.embl`: base substitution reconstruction in EMBL format.
        * `gubbins.summary_of_snp_distribution.vcf`: VCF file summarising the distribution of SNPs.
        * `gubbins.per_branch_statistics.csv`: per branch reporting of the base substitutions inside and outside recombination events.
        * `gubbins.filtered_polymorphic_sites.fasta`: FASTA format alignment of filtered polymorphic sites used to generate the phylogeny in the final iteration.
        * `gubbins.filtered_polymorphic_sites.phylip`: phylip format alignment of filtered polymorphic sites used to generate the phylogeny in the final iteration.
        * `gubbins.final_tree.tree`: newick format file containing the final phylogeny.
        * `gubbins.node_labelled.tre`: newick format file containing the final tree with nodes annotated for comparison with FastML's sequence reconstruction and the per branch statistics CSV file.
  
      * It finally runs `snp-sites`, with the `-c` option, using `gubbins.filtered_polymorphic_sites.fasta` as input, outputting the file `clean.core.aln`. This file will consist, therefore, in an **SNP alignment** of the **core genome** with **both variant and invariant sites**, but this time **with no recombinant sites**.
  
    * Creates a script called `_03_run_gubbins.sh` that will run `_03_gubbins.sh` as a slurm job.

    > **After running `lablog`, run `_00_snippy.sh`, `_01_snippy_core.sh`, `_02_phylo_aln.sh` and `_03_run_gubbins.sh` sequentially.**
  
  * `05-iqtree`: this folder contains a `lablog` file that does the following:
    * Creates a subfolder called `logs`.
  
    * Creates a script called `_00_iqtreemfp.sh` which will run iqtree on the `phylo.aln` file from the `04-snippy` folder. Since we are not sure which model is appropriate for our data, we use **ModelFinder** to determine the best-fit model (by means of the `-m MFP` option).
  
    * Creates a script called `_01_iqtreeall.sh` which will run iqtree on the `phylo.aln` file from the `04-snippy` folder, this time using the most appropriate model (**the best-fit model is automatically picked from the .log file obtained after running `_00_iqtreemfp.sh`. Running the lablog a second time will autofill de `-m` option in `_01_iqtreeall.sh`.**). Iqtree will be run with a specific number of CPU cores (by means of the `-T` option, 20 by default) and 1000 bootstrap replicates by default (by means of the `-B` option). The resulting files will be saved with the `phylo.iqtree.bootstrap` prefix (option `-pre`).

> **Ignore this if you DID NOT remove recombinant sites from the initial SNP alignment from snippy**:<br><br>
> After running the `_03_run_gubbins.sh` script, you'll get a file called `clean.core.aln`, equivalent to `phylo.aln` but with no recombinant sites this time. You'll have to repeat the previous procedure in order to obtain the best phylogenetic tree from this file.
> * Create a new folder called `06-iqtree-gubbins` using `mkdir`.
> * Copy the `lablog` file from 05-iqtree into this new subfolder using `cp`.
> * Modify this `lablog` file so that the input used for `_00_iqtreemfp.sh` and `_01_iqtreeall.sh` is not `phylo.aln`, but `clean.core.aln`.
> * Run this `lablog` file: `bash lablog`.
> * Run the `_00_iqtreemfp.sh` and `_01_iqtreeall.sh` files as indicated previously.
> 

  * `99-stats`: this folder contains a `lablog` file that does the following:
    * Creates a symbolic link to the reference file used by snippy, which will be saved as `reference.fna`.

        >[!WARNING]
        >The reference file is **not** filled in automatically. Before running this `lablog` file, please add manually the name of the reference file in **line 3**.
    
    * Creates a subfolder called `logs`.
  
    * Runs `samtools flagstat` on the `.bam` files obtained from snippy for each sample, and extracts the percentage of mapped reads against the reference, saving the output in a file called `mapping_stats.txt`, a tab-separated file which will show this percentage next to each sample.
  
        >Example of the output shown by `samtools flagstat`:
        >```
        >$ samtools flagstat snps.bam
        >
        >9583304 + 0 in total (QC-passed reads + QC-failed reads)
        >5 + 0 secondary
        >0 + 0 supplementary
        >0 + 0 duplicates
        >7589985 + 0 mapped (79.20% : N/A)
        >9505556 + 0 paired in sequencing
        >4752651 + 0 read1
        >4752905 + 0 read2
        >7512492 + 0 properly paired (79.03% : N/A)
        >7539658 + 0 with itself and mate mapped
        >5232 + 0 singletons (0.06% : N/A)
        >6184 + 0 with mate mapped to a different chr
        >5045 + 0 with mate mapped to a different chr (mapQ>=5)
        >```
        > We would be interested, in this case, in extracting the 79.20% percentage.
        >

    * Creates a script called `_00_wgsmetrics.sh`, which will run `picard CollectWgsMetrics` on the `.bam` files obtained from snippy. This function collects different metrics regarding the coverage of the samples being analysed against the corresponding reference. This includes information such as the reference genome size, the mean coverage in bases of the genome territory, the fraction of bases that attained at least 10X sequence coverage, etc. This procedure is done for each sample separately, creating a file ending in `_collect_wgs_metrics.txt`.
  
    * Creates a script called `_01_gather_wgs_metrics.sh`, which generates:
      * A file called `wgs_metrics_all.txt`, which takes the information from each .txt report from picard and collects it as a whole.
  
      * A file called `wgs_metrics_all_filtered.txt`, which contains certain columns from the previous file, which are of more relevance for the researcher.
  
    * Creates a script called `_02_generate_summary.sh`, which adds a header to the `mapping_stats.txt` filef, combines the information from this file with `wgs_metrics_all.txt` and selects specific columns from the merged data, generating a final file called `mapping_stats_summary.txt`.

    * Creates a final script called `_03_variants_stats.sh`, which reads the .log file obtained after running the `_01_snippy_core.sh` script from `04-snippy` and then extracts the number of SNPs, deletions, insertions and heterozygous sites. This information is shown in a file called `variants_stats.txt`, so that these variants are written for each sample separated by semicolons.

Once you have gone through all these folders and run all the required scripts, you should go to the `RESULTS` folder and have a file called `lablog_snippy_results` (usually along with `lablog_assembly_results`, `lablog_plasmidid_results` and `lablog_characterization_results`). The `lablog` file corresponding to the Snippy service does the following after being run:
  * Creates a folder ending in *entrega01*.
  
  * Creates, inside this folder, a subfolder called `snp`.
  * Inside this subfolder, it creates symbolic links to the following files:
    * `phylo.iqtree.bootstrap.treefile` (saved as `phylo.iqtree.bootstrap.nwk`) from `05-iqtree`.
    * `variants_stats.txt` from `99-stats`.
    * `mapping_stats_summary.txt` from `99-stats`.
    * `wgs_metrics_all_filtered.txt` from `99-stats`.

> If you generated another .treefile file from `clean.core.aln`, please create the corresponding symlink from the `06-iqtree-gubbins` folder.

## What should I do after I've run all the necessary scripts?

Once we are done with the service (including the assembly, characterization and plasmidid procedures, since this is the usual case), we'll have to review the results from each procedure and add all the relevant information into an Excel file. For this, we use a **template** that you can find [**here**](https://docs.google.com/spreadsheets/d/1m_hnCGNgtWcoJAjs_91BkmfJn6CNr4yO/edit?usp=drive_link&ouid=108428245306738036878&rtpof=true&sd=true).

In this Excel file, we can find the following sheets:
* **summary**: this sheet contains information regarding the assembly performed previously (check the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) page), Kmerfinder, the MLST profile identified for the samples and the mapping procedure done against the reference used for snippy.
  * For the **MLST profile**, please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) page.
  * For the **Kmerfinder** columns `07-kmerfinder_best_hit_# Assembly`, `07-kmerfinder_best_hit_Accession Number` and `07-kmerfinder_best_hit_Description`, please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) page.
  * The **MAPPING** columns are relative to snippy. **Please modify the header and indicate the correct reference used**:
    * `%mapping`: this information is obtained from the `mapping_stats.txt` file from `99-stats`.
    * `Depth of coverage`: this information is obtained from the `wgs_metrics_all_filtered.txt` file from `99-stats` (column `MEAN_COVERAGE`).
    * `coverage > 10x`: this information is obtained from the `wgs_metrics_all_filtered.txt` file from `99-stats` (column `PCT_10X`).
    * `Variants (SNP;DEL;INS;HET)`: this information is obtained from the `variants_stats.txt` file from `99-stats`.
  * The **ASSEMBLY** columns are relative to the assembly service. Please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) page.
* **snpmatrix_all_pos**: please go to the [**How can I obtain the SNP matrices for the Excel summary file?**](#how-can-i-obtain-the-snp-matrices-for-the-excel-summary-file) section. This sheet is filled in after loading **`phylo.aln.fasta`** into **MEGA**.
* **snpmatrix_all_pos_pairs**: please go to the [**How can I obtain the SNP matrices for the Excel summary file?**](#how-can-i-obtain-the-snp-matrices-for-the-excel-summary-file) section. This sheet is filled in after loading **`phylo.aln.fasta`** into **MEGA**.
* **snpmatrix_core**: please go to the [**How can I obtain the SNP matrices for the Excel summary file?**](#how-can-i-obtain-the-snp-matrices-for-the-excel-summary-file) section. This sheet is filled in after loading **`clean.core.aln.fasta`** into **MEGA**.
* **snpmatrix_core_pairs**: please go to the [**How can I obtain the SNP matrices for the Excel summary file?**](#how-can-i-obtain-the-snp-matrices-for-the-excel-summary-file) section. This sheet is filled in after loading **`clean.core.aln.fasta`** into **MEGA**.
* **plasmids**: please check the [**PlasmidID service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) and the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* **virulence**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* **Resistance result**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* **AMRFinderPlus Resistance result**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* **MLVA**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).

Fill in the Excel template following the previous instructions, name it according to the structure `summary_outbreak_species.xlsx`, and then do the following:
* Being logged in your Google account, go to the **buisciii_shared** Drive folder.
* Go to **ALERTS**.
* Create a new subfolder called after the organism of interest followed by the date in which the service was requested. For example, if the service that we are carrying out is relative to *Brucella melitensis* and the service was requested on 28/10/2024, the new folder will have this name: `BMELITENSIS_20241028`.
* Go inside this subfolder and store the Excel report in here.

Once the Excel file has been completed with all the necessary information and has been updated into the corresponding Drive folder, **save a copy of this Excel file** both inside the `RESULTS/DATE_entrega01` folder and the `ANALYSIS` folder of the service. You can then proceed with the copy of the service to `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` and `/data/ucct/bi/sftp`.

### How can I obtain the SNP matrices for the Excel summary file?

As you saw before, some sheets in the Excel summary file are filled in after loading phylo.aln or clean.core.aln into MEGA, with the objective of obtaining the corresponding SNP matrix for the samples. In this subsection, you'll see how to obtain this matrix from these files.

First of all, phylo.aln and clean.core.aln have to be in .FASTA format before the submission into MEGA. Run the following commands:

```
cp phylo.aln phylo.aln.fasta
cp clean.core.aln clean.core.aln.fasta
```
Now, we're ready to run MEGA.

> **MEGA is run in your local PC**, not in the HPC. Therefore, if you don't have MEGA installed, please download it now and install it. To do so, go to the [**MEGA webpage**](http://www.megasoftware.net/), download the MEGA `.deb` package and install it this way:
>```
>sudo dpkg -i path/to/the/deb/package
>```
>
>You should be able to see MEGA among your installed applications. Please run it.

After running MEGA, you should see this screen:

![MEGA-screen](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/MEGA-screen.png)

To load `phylo.aln.fasta` or `clean.core.aln.fasta` into MEGA, follow these steps:

1. Click on **DISTANCE** > **Compute Pairwise Distances**
2. On the emerging window, please go to the directory in which **`phylo.aln.fasta`** or **`clean.core.aln.fasta`** is located.
3. On the next window, click on **Nucleotide sequences** and indicate "N" on the **Missing Data** box.
4. Reply **NO** to the emerging window asking *Protein-coding nucleotide sequence data?*
5. Reply **YES** to the emerging window asking *Would you like to use the currently active data?*
6. Select these options on the next window and click on OK:
![MEGA-setup](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/MEGA-setup.png)
7. You'll see a matrix like this one:
![MEGA-matrix](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/MEGA-matrix.png)
8. Click on the superior ***Export distances as Excel file*** button.
9. On the emerging window, select **0 decimal places**, export as **matrix** and lower left matrix. You'll get this matrix as an Excel file. Copy this matrix and paste it on the **`snpmatrix_all_pos`** sheet from the summary Excel file:

    ![MEGA-export](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/MEGA-export.png)

10.  Repeat this procedure, but this time select **Column** as export type. You'll get an Excel file; copy it and paste it on the **`snpmatrix_all_pos_pairs`** sheet from the summary Excel file.
11.  Repeat the previous steps for the **`clean.core.aln.fasta`** file. Repeat steps 9 and 10 with this file, and copy the Excel results from MEGA into the **`snpmatrix_core`** and **`snpmatrix_core_pairs`** sheets from the Excel summary file.

### What if I want to check what the tree obtained after `iqtree` looks like?

As indicated previously, after running `iqtree` we should obtain a `.treefile` file that will be then stored in the corresponding subfolder from `RESULTS` for the researchers to check. 

However, it is recommendable to **visualise the tree** we just generated in order to see if your results make sense or with the objective of analysing the phylogenetic relationships existing among the samples from the service.

There are several **tree** **visualisers**, but the one that will be explained in this document, and that is used the most within the Unit for this purpose, is [**iTOL**](https://itol.embl.de/). **iTOL** is an online tool for the **display**, **annotation** and **management** of **phylogenetic trees**. It allows the user to manage and visualise trees directly in the browser, annotating them at the same time with various datasets.

Let's say you already have a `.treefile` file from `iqtree` and you want to visualise it in iTOL. To do so, follow these steps:
1. Create an account in iTOL if you don't have one. Otherwise, please log in with your credentials.
2. Once logged in, click on "**My Trees**".
3. In this page, you'll see a list of all the trees you have uploaded into iTOL. To upload a new tree, click on "**Tree upload**" and either drop the `.treefile` file into the emerging box or select it on an emerging window.
4. Once uploaded, it is very recommendable to rename the tree so that you can identify it easily later. Usually, we name the uploads in iTOL according to the service ID and indicating if they were generated from `phylo.aln` or `clean.core.aln`, to wit, before or after running gubbins to filter for recombinant sites.
5. Once renamed, click on the tree name. You'll see a first version of your tree, but we should make a few adjustments before reviewing it properly:
   1. On the **control panel**, click on **Advanced** -> **Midpoint root**.
   2. **Advanced** -> **Bootstraps / metadata** -> **Display**.
      1. Click on **Text**, Font **14** px.
   3. **Advanced** -> **Tree scale box** -> **Display**.
   4. **Advanced** -> **Leaf node symbols** -> **Display**.
   5. **Advanced** -> **Internal node symbols** -> **Display**.
6. If you want to export the tree, you can click on the **control panel** and then **Export**. You can export your tree in SVG, EPS, PDF, PNG, newick, phyloXML and NEXUS formats.

Once you're done, your tree will be stored in your account with the changes you made, so you'll be able to check it whenever you like.

### What if I want to check the size of `phylo.aln`?

Please run:
```
awk 'BEGIN{FS="[> ]"} /^>/{val=$2;next} {print val,length($0)}' phylo.aln
```

### What if I need to remove complex variants?

In some situations, it might be appropriate to remove **complex variants** before generating the `core.full.aln` file when running `_01_snippy_core.sh`. If this is the case, run the following (these commands are also in the `lablog` file from `04-snippy`):
```
grep \"complex\" ./*/snps.vcf | cut -f 1,2,4,5 | cut -d \":\" -f 2 | sort -u | awk '{pos1=\$2; len_ref=length(\$3); printf \"%s\t%s\t%s\n\", \$1, pos1-1, pos1+len_ref+1}' | grep -v \"^#\" > mask_complex_variants.bed
```
As you can see, this command generates a .bed file that indicates the positions of all complex variants. This will be used by the `snippy-core` function next.

```
scratch_dir=$(echo $PWD | sed 's/\/data\/ucct\/bi\/scratch_tmp/\/scratch/g')
srun --chdir ${scratch_dir} --output logs/SNIPPY_CORE.%j.log --job-name SNIPPY --cpus-per-task 5 --mem 49152 --partition short_idx --time 02:00:00 env - PATH="$PATH" singularity exec -B ${scratch_dir}/../../../ /data/ucct/bi/pipelines/singularity-images/snippy:4.6.0--hdfd78af_4 snippy-core --debug --mask ./mask_complex_variants.bed --mask-char 'N' --ref '../../../REFERENCES/XXX' $(cat ../samples_id.txt | xargs)" >> _01_snippy_core.sh
```
After this, run `_01_snippy_core.sh`, and you'll get the SNP alignment with no complex variants.

>[!WARNING]
>Indicate manually the reference used for snippy before running `_01_snippy_core.sh` (replace `XXX` with the reference filename). You'll have to modify it using `nano`, `vim` or another editing tool.

### What if I need to remove low coverage variants?

In some situations, it might be appropriate to remove low coverage variants that may be false positives, and this should be done before generating the `core.full.aln` file when running `_01_snippy_core.sh`. If this is the case, run the following (these commands are also in the `lablog` file from `04-snippy`):

```
awk -F'\t' '/^NZ_/ {split($9, format, ":"); split($10, values, ":"); for (i in format) if (format[i] == "DP" && values[i] < 10) print $1, $2 - 1, $2}' OFS='\t' ./*/snps.vcf > mask_low_coverage_variants.bed
```
As you can see, this command generates a .bed file that indicates the positions of all low-coverage variants. This will be used by the `snippy-core` function next.

```
scratch_dir=$(echo $PWD | sed 's/\/data\/ucct\/bi\/scratch_tmp/\/scratch/g')
srun --chdir ${scratch_dir} --output logs/SNIPPY_CORE.%j.log --job-name SNIPPY --cpus-per-task 5 --mem 49152 --partition short_idx --time 02:00:00 env - PATH="$PATH" singularity exec -B ${scratch_dir}/../../../ /data/ucct/bi/pipelines/singularity-images/snippy:4.6.0--hdfd78af_4 snippy-core --debug --mask ./mask_low_coverage_variants.bed --mask-char 'N' --ref '../../../REFERENCES/XXX' $(cat ../samples_id.txt | xargs)" >> _01_snippy_core.sh
```

After this, run `_01_snippy_core.sh`, and you'll get the SNP alignment with no low-coverage variants.

**If you need to remove both complex and low-coverage variants at the same time, merge both .bed files and run snippy-core as indicated before, indicating the correct .bed file in the `--mask` option.**

### What if I want to check whether a variant is a false positive?

As indicated before, snippy is run so that the minimum number of reads covering a site to be considered as a variant is 9. Sometimes, there might be a low depth of coverage of a certain nucleotide for a specific genomic position.

In some situations, for example when two samples belong to the same Strain Type, it might be a good idea to check **whether the number of SNPs identified in the SNP matrix for these two samples is real or not**. Depending on the real number of variants, it could also be useful to then discard low-coverage and/or complex variants as well.

How can we determine whether a variant is real or not? This is done with **Integrative Genomics Viewer** (**IGV**), which you can download [**here**](https://igv.org/doc/desktop/#DownloadPage/).

>[!WARNING]
>Download IGV in your **local computer**, not in the HPC.

For this process, you should check the file called `core.tab` obtained after running `snippy-core`. This file contains several columns regarding the identified variants, which correspond to the reference, the position of the variant and the bases that were identified in the reference and the samples.

Let's use a example. Let's say we have two samples, called 001 and 002. After running the `_00_snippy.sh`, `_01_snippy_core.sh` and `_02_phylo_aln.sh` scripts, we obtained the `phylo.aln` file that was described previously. When loading it into MEGA, we saw that there were 20 SNPs between these two samples. When loading `clean.core.aln` into MEGA, these 20 SNPs were reduced to 11 SNPs. Are these 11 SNPs real? Do any of the 20 initial SNPs have low depth of coverage? Are any of these SNPs complex? Let's check this out by means of the `core.tab` file and IGV.

First, let's open `core.tab`, and let's filter the data contained in this file so that we see only those positions of the reference genome for which the base sequenced for samples 001 and 002 is different. To do so, we should run this command in the terminal:

```
awk '$4 != $5' core.tab | cut -f 1-5
```

By this command, we are selecting the first three columns of the file (reference ID, position and reference base), and then the columns corresponding to samples 001 and 002. However, only those rows for which the base is different for samples 001 and 002 will be shown (that's what `awk` is doing). Since `core.tab` is obtained before running gubbins, we'll see in this case the 20 initial SNPs that exist between both samples:

```
CHR	POS	REF	001	002
NZ_CP076401.1	303947	G	T	G
NZ_CP076401.1	540673	T	T	A
NZ_CP076401.1	636316	T	T	C
NZ_CP076401.1	672300	C	C	T
NZ_CP076401.1	1012282	G	G	A
NZ_CP076401.1	1508262	T	T	A
NZ_CP076401.1	1589627	A	A	G
NZ_CP076401.1	2080870	G	G	T
NZ_CP076401.1	2420727	A	A	T
NZ_CP076401.1	2420728	T	T	A
NZ_CP076401.1	2420729	A	A	T
NZ_CP076401.1	2420730	T	T	A
NZ_CP076401.1	2420731	A	A	T
NZ_CP076401.1	2420732	C	C	A
NZ_CP076401.1	2608781	A	A	G
NZ_CP076401.1	2612437	C	C	T
NZ_CP076401.1	3440421	G	A	G
NZ_CP076401.1	3552311	T	C	T
NZ_CP076401.1	3675041	C	T	C
NZ_CP076401.1	3805781	G	C	G
```

Now, you should copy this output and paste it into an **Excel** sheet. Then, create two new columns: **TRUE/FALSE** and **REASON**, since **we will check which of these 20 SNPs aren't real variants and why**.

Once you're sheet is ready, and considering you've already installed IGV in your **local computer**, run IGV. You'll see this screen:

![IGV-screenshot](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/IGV-screenshot.png)

Now, follow these steps. First, you will load the reference genome and then you'll load the `snps.bam` files you obtained previoulsy after running `snippy` for each sample:
1. Go to **Genomes** -> **Load Genome from File**. Select the file you used previously as reference genome.
2. Go to **File** -> **Load from File**. Select the `snps.bam` file from one sample. Then, click again on the same button and select the same file from the other sample.
3. Given your Excel file, copy the position of the first variant and paste it in the box that is next to the **Go** button.

After following these steps, you'll see something like this (after zooming in):

#### CASE 1: TRUE VARIANT

![IGV-screenshot-true-variant](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/IGV-screenshot-true-variant.png)

Using your mouse, you can search for the specific position that is specified in the `core.tab` file and click on the superior boxes (one track corresponds to one sample and the other track belongs to the other sample). In our case, one of the boxes is grey and the other one is red for the same position. If we click on these boxes, we'll see these little windows:

![TP-comparison](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/TP-comparison.png)

In this case, mostly all reads have the same base for this specific location, and this happens for both samples. The called alleles have a good depth of coverage, so we can infer that this is a **TRUE VARIANT**, since one of the samples has a lot of reads for a base that is different from the reference base and, since both samples have a good depth of coverage, we can be sure that this position really has a different base for each sample, so the difference reported by `core.tab` is real.

 We should thereby indicate so in the Excel file that we created before.

#### CASE 2: FALSE VARIANT DUE TO LOW DEPTH OF COVERAGE

Now, using the same example as before, let's check another case you might come across, which is when one of the called variants has low depth of coverage, making it a false positive. Check this case:

![IGV-screenshot-low-depth](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/IGV-screenshot-low-depth.png)

On position 3440421, you can see an A for both samples. However, let's check the number of reads that have that base for each sample:

![LD-comparison](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/LD-comparison.png)

As you can tell, one of the samples has only 9 reads covering this position, and all these reads have an A. Then, in theory, this position shouldn't be considered a position for which both samples have different bases; both have the same base, but this is not what `core.tab` says. 

Snippy is configured so that the minimum number of reads covering a site to be considered as a variant is 9, so snippy did not consider the first sample to have a variant (even though we can see that it does) and, when checking `core.tab`, this sample is indicated to have a G (reference base) in this position, when in reality no reads have a G for this sample.

Both samples do indeed have an A, so both have a substitution for this position in relation to the reference genome. This is a real variant against the reference, but these samples do not really differ in their base for this position, as indicated by `core.tab`. Therefore, what is indicated by `core.tab` is FALSE; there is no real difference between both samples since both do have the same allele.

In our Excel file, we should indicate this, apart from saying that for one of the samples the allele has low depth of coverage. This is why snippy didn't consider it a real variant and indicated that this sample has the reference base for this position in `core.tab`, when it is obvious that it does not.

#### CASE 3: FALSE VARIANT DUE TO CLOSE INDELS

In some cases, a variant might not be true because there are **insertions/deletions** close to the position that was reported. For example, let's check this image:

![FP-indel-comparison](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/FP-indel-comparison.png)

For a specific position, both samples have the same base (an A, which is also the reference base) for almost all reads, but right next to this position there is an insertion. This will affect the variant calling, so false differences between the samples will be identified, which is the case in this situation.

Furthermore, these indels are responsible for the appearance of **complex variants**. Indeed, if we check this position in one of the samples being compared, specifically in its `snps.vcf` file, we'll see this position is associated with a complex variant. 

If you come across a situation like this, write FALSE on your Excel sheet and indicate there are close insertions or deletions next to the position of interest.

#### CASE 4: RECOMBINANT SITES

Sometimes, a certain position reported by `core.tab` will have different bases for each sample, but these samples will have reads will a specific base and other reads with another sample at the same time. Let's see an example given our two samples from before:

![recombinant-site](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/recombinant-site.png)

As you can see, both samples have reads with a C and reads with a G at the same position, so calling a base in this case gets more difficult. This is a **recombinant site** that **Gubbins** will discard so, even though there is a real difference between both samples, this position will not be considered when creating `clean.core.aln`, so we should not take it into account.

If you come across a situation like this, write TRUE on your Excel file, but indicate that this position will be discarded by Gubbins since it's a recombinant site.

---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion. To do this, execute the **finish** module of buisciii-tools. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

    $ buisciii --log-file SRVCNMXXX.X.finish.log finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (in **Snippy**, this folder is `02-preprocessing`). Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ buisciii --log-file SRVCNMXXX.X.tool.log bioinfo-doc SRVCNMXXX.X

Remember to save the logs with the corresponding name (i.e. `SRVCNMXXX.X.service_info.log` or `SRVCNMXXX.X.delivery.log`).

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

>[!WARNING]
> When performing an **outbreak** service (as mentioned before, this Snippy service is usually not done alone, but along with the assembly, plasmidid and characterization procedures), the delivery message is in general too long to be included as a .txt file during the delivery procedure. Therefore, for this kind of services, reply the following when these questions are asked on the terminal:
>1. Do you wish to provide a text file for delivery notes?: type n.
>2. Write some delivery notes: leave it blank, by pressing Enter.
>3. Do you want to add some delivery notes to the e-mail?: type n.
>4. Do you want to send e-mail automatically?: type n.
>
>The service_info and delivery .pdf files will have been created but the e-mail won't have been sent. This will be done manually by sending the service report that you can see in the last section of this manual.

Lastly, once the service has been delivered and the e-mail has been sent, remember to remove all the files related to this service from `scratch_tmp`:

    $ buisciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
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