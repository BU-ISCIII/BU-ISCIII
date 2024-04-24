# How to perform a PikaVirus Service

This is a brief tutorial on how to perform a Pikavirus Service as a member of the ISCIII's Bioinformatics Unit! PikaVirus is a associated with _Viral: Detection and characterization of viral genomes within metagenomic data_ service from service catalog.

[PikaVirus](https://github.com/BU-ISCIII/PikaVirus) is a bioinformatics best-practise analysis pipeline for metagenomic analysis following a new approach, based on eliminatory k-mer analysis, followed by assembly and posterior contig-binning. This service will allow us to identify the viral species present in a sample.

Let's get started with the service. When performing a PikaVirus service, remember that the typical acronym is `VIRAL-DISCOVERY`, but this may differ depending on the service.

First of all, follow the [first steps to create a service](/link/to/tools/and/iskylims/TODO). Once the `new-service` is finished, you'll have a new folder with 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP` (as explained [here](https://github.com/BU-ISCIII/BU-ISCIII/wiki/bioinformatics#33-services_and_collaborations)). We should check the following folders before going any further:

- `DOC`: This folder should contains a `hpc_slurm_pikavirus.config` file with the specific configuration for nf-core/mag in our HPC and some custom parameters like databases paths.
- `RAW`: Check that the number of files contained within the RAW folder is equal to the number of samples specified in [iskyLIMS](https://iskylims.isciii.es/) x 2, in the case that they are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:

- `lablog_pikavirus`: an executable file that creates the 00-reads folder, moves inside, creates symbolic links to the reads renaming them and renames the `ANALYSIS01_PIKAVIRUS` folder.
- `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.
- `ANALYSIS01_PIKAVIRUS`: Folder with the main PikaVirus analysis files.

Now we can execute the lablog:

```bash
bash lablog_pikavirus
```

> [!WARNING]
> If PikaVirus is not the only analysis in your service, don't forget to run the other `lablogs` before the next steps.

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch` as explained [here](/link/to/tools/and/iskylims/TODO)

Once this function is finished, we should go into the `scratch_tmp` folder and the specific `ANALYSIS01_PIKAVIRUS` folder associated with our service. Once we're inside, we will see the following folders/files:

- `lablog`: will create symbolic links to `00-reads` and the `samples_id.txt` file, creates `pikavirus.sbatch`, `_01_nf_pikavirus.sh` and `samplesheet.csv` files.

Let's execute the `lablog` file:

```bash
bash lablog
```

Now, we should check we've loaded all the needed dependencies and perform the metagenomic analysis:

```bash
module load Nextflow/21.10.6 singularity
bash _01_nf_pikavirus.sh
```

After this, the analysis will start, and we'll be able to check the status of the process with:

```bash
tail -f DATE_pikavirus01.log
```

Once checked everything has finished OK in the `DATE_pikavirus01.log`, within the `DATE_ANALYSIS01_PIKAVIRUS` folder, we'll have the following content:

- `01-PikaVirus-results/`: Results of the PikaVirus pipeline.
  - `multiqc_report.html`: MultiQC's HTML report of the PikaVirus pipeline.
  - `mash_results`: Mash results for each sample, we would need to check them if there is any problem with the hits.
  - `<sample_name>`: Results specific for the sample (quality control, multiQC, coverage...)
  - `<sample_name>_results.html`: HTML results with the stats for the sample (hits found, coverage, stats...)
  - `all_samples_virus_table.tsv`: Table with the results for all the samples and the stats in .tsv format.

The file `all_samples_virus_table.tsv` usually contains some duplicated reads and phages. In order to clean the results in this table we should run:

```bash
grep -v 'genome' all_samples_virus_table.tsv | grep -v 'phage' > all_samples_virus_table_filtered.tsv
```

Then convert this table to `.xlsx` if this process is not included in the `RESULTS`'s `labglog`.

In this service we should check:

- `01-PikaVirus-results/multiqc_report.html`: FastQC/fastp reports to assess the good/bad quality of the reads, the presence of adaptaers, etc...
- `all_samples_virus_table.tsv`: Check that the species are the spected and that they contain enough coverage. Those virus covered to >50% at a depth of 10X, can be used as reference genome for `nf-core/viralrecon` for further analysis.
