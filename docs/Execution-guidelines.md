# Pipeline execution guidelines

BU-ISCIII currently implements its pipelines using nextflow and singularity; however some of our pipelines are not migrated yet or some specific analysis don't have the enough entity to make a automatic pipeline (for example while we are testing something).
In these cases we use a standardized organization for the analysis in order to be able to understand an replicate the execution and follow the results (all analysis are performed in ANALYSIS folder following [[bioinformatics]] structure description):

- **samples_id.txt file:** contains the sample identifiers for all the samples being analyzed in the project. Usually sample names are included in the fastq files with this format: ```{sample_name}_S45_R1_001.fastq.gz```. One easy way for setting samples_id.txt file is locating yourself in ANALYSIS folder and executing:

```Bash
 find ../RAW_NC -name "*.fastq.gz" | cut -d "/" -f 3 | cut -d "_" -f 1 | sort -u > samples_id.txt
```

> NOTE: This is just an example fastq files names can have multiple formats, and there are a bunch of different ways to generate samples_id file.

- **00-reads:** we use this folder as starting point for our analysis. It is useful for simplify posterior steps to rename fastq files to just ```{sample_name}_R{1,2}.fastq.gz```. As we don't want to rename the original files we use this folder to make **symbolic links** with the new names. We can use this command for this, locate yourself inside 00-reads for easier use of ln command:

```Bash
cat ../samples_id.txt | xargs -I % echo "ln -s ../../RAW/%_*R1*.fastq.gz" %_R1.fastq.gz" | bash
cat ../samples_id.txt | xargs -I % echo "ln -s ../../RAW/%_*R2*.fastq.gz" %_R2.fastq.gz" | bash
```

- **01-fastQC, 02-preprocessing:** next steps of the pipeline will follow having each one its own folder. Each folder must have a **lablog** file and the **scripts** needed for regenerate the results stored in that folder. Also a **logs folder** is created where all generated logs are stored (You **MUST** ensure that logs are saved, if you are using ISCIII's HPC you must save *.oXXXX files, but if you are executing something directly in the shell you must redirect the standard output to a log file an store it in logs folder).

- **lablog:** this file is located in each folder of the analysis, it indicates the commands that must be run for replicate the results found in its folder. Most of the times lablog does not contain the commands for running the analysis, instead it runs commands for generating the necessary scripts for performing the analysis. For example necessary orders for iterating among all the samples in the analysis will be set in this file. And of course it will describe all the steps needed for the analysis. Let's see an example, the lablog file contained in 01-fastQC folder will contain the fastqc command for all the samples in the project.

Imagine samples_id.txt looks like this:

```
sample1
sample2
sample3
```

lablog file inside 01-fastQC folder would look like this:

```Bash
## This line iterates using samples_id.txt file and executes the xargs part one time per line in the samples_id.txt file. In each iteration "%" works as a variable that contains each value in each line (sample1, sample2,...)
cat ../samples_id.txt | xargs -I % echo "mkdir %;fastqc -o % ../00-reads/%_R1.fastq.gz ../00-reads/%_R2.fastq.gz" >> _00_rawfastqc.sh
## If you feel more confortable with traditional loops the equivalent way using while would be:
cat ../samples_id.txt | while read in; do echo "mkdir "$in";fastqc -o "$in" ../00-reads/"$in"_R1.fastq.gz ../00-reads/"$in"_R2.fastq.gz"; done >> _00_rawfastqc.sh
## In addition in this step we incorporate an order for unziping fastqc results. Same as above we iterate though samples id and we will create 3 unzip commands one for each sample.
cat ../samples_id.txt | xargs -I % echo "cd % ; unzip \*.zip; cd .." > _01_unzip.sh
```

When we execute lablog file: ```bash lablog``` we generate two scripts that will look like this:
**_00_rawfastqc.sh**

```Bash
mkdir sample1;fastqc -o sample1 ../00-reads/sample1_R1.fastq.gz ../00-reads/sample1_R2.fastq.gz
mkdir sample2;fastqc -o sample2 ../00-reads/sample2_R1.fastq.gz ../00-reads/sample2_R2.fastq.gz
mkdir sample3;fastqc -o sample3 ../00-reads/sample3_R1.fastq.gz ../00-reads/sample3_R2.fastq.gz
```

**_01_unzip.sh**

```Bash
cd sample1 ; unzip \*.zip; cd ..
cd sample2 ; unzip \*.zip; cd ..
cd sample3 ; unzip \*.zip; cd ..
```
