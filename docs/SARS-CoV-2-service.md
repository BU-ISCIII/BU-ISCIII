# Viralrecon SARS CoV 2 service description

1. Create the service using `buiscii-tools` and move it to `scratch_tmp`.
2. Go to `ANALYSIS`:
   1. Create `samples_ref.txt` (if needed, replace `NC_XXXXXX` and `Human` with the reference and host requested):

      ```bash
      1. sed 's/$/\tNC_045512.2\tHuman/' samples_id.txt > samples_ref.txt
      ```

3. Run `bash lablog`. This should create a `_01_run` for each reference (in most cases, only 1).
4. Execute `Module load Nextflow singularity`
    * **Note:** As with any viralrecon analysis, make sure to use the latest [pangolin](https://github.com/cov-lineages/pangolin/releases) and [Nextclade](https://github.com/nextstrain/nextclade_data/blob/release/CHANGELOG.md) database and software versions. You may find the latter in the [Galaxy's singularity depot](https://depot.galaxyproject.org/singularity/)
5. Run `bash _01_run_NC_XXXXXX_viralrecon.sh`. To execute the second, wait for the first to finish.
6. While the first process is running, you can go to `_0X_MAG` and run `bash lablog` to generate the kraken tables.
   1. Once this process is complete, enter `99-stats/` and load the MultiQC module `Module load MultiQC` and run `lablog bash lablog`.

Once the `viralrecon` processes are completed, generate results by executing the remaining scripts in `ANALYSIS_0X_/`:

1. To run `_02_create_run_percentage_Ns.sh` and `_03_run_percentage_Ns.sh`, you will need to activate the proper environment with `conda activate python3`. It's recommended to run `module purge` beforehand.
2. Sequentially run `bash _02_create_run_percentage_Ns.sh`, `bash _03_run_percentage_Ns.sh`, and `bash _04_create_stats_table.sh`.
3. To run `_05_create_stats_assembly.sh`, execute `module load R/4.2.1`; You may need to configure the R environment if it's your first time:
   1. Create the file `/home/user/.Renviron` containing the following text: `R_LIBS_USER=/data/bi/pipelines/R-lib/`.

After this, results can be grouped in the `RESULTS` folder automatically following these steps:

1. Run lablog `bash viralrecon_results`.
2. Activate the appropriate conda environment with `conda activate viralrecon_report`.
3. Go to `DATE-entrega0X` and run the commands `bash _01_generate_excel_files.sh` and `bash _02_clean_folders.sh`.

### Common errors while running the service

Make sure that `samples_ref` file does not contain any spaces as it's read as a tab-separated file. You can use `cat -A samples_ref.txt` to ensure this (`\t` is shown as `^I` in this case).

When there's a mix of full-numerical and strings in sample IDs (e.g. `87439.fastq.gz` and `SARS_01.fastq.gz`) the pipeline may crush in `MULTIQC` step. This is caused because there's a bug with MultiQC ([MultiQC issue](https://github.com/nf-core/viralrecon/issues/345)) that can be temporarily fixed by adding any non-numerical character to the sample IDs.

### Handling out results SARS-CoV-2 service

1. Read .log from Nextflow saved when nohub is performed

e.g :

```
    cd 20220209_ANALYSIS01_AMPLICONS_HUMAN/
    cat 20220209_mapping01.log 
```

To find the errors in the log do:

e.g :

```
    grep 'termin' 20220209_mapping01.log    
    [61/30f1aa] NOTE: Process `CUTADAPT (220648)` terminated with an error exit status (134) -- Error is ignored
    [83/4ebcf1] NOTE: Process `CUTADAPT (220649)` terminated with an error exit status (134) -- Error is ignored
    [39/1091c2] NOTE: Process `IVAR_CONSENSUS (220600)` terminated with an error exit status (1) -- Error is ignored
    [5e/0fd305] NOTE: Process `BCFTOOLS_CONSENSUS (220600)` terminated with an error exit status (1) -- Error is ignored
    [65/2c4848] NOTE: Process `IVAR_CONSENSUS (220603)` terminated with an error exit status (1) -- Error is ignored
    [f9/b2e511] NOTE: Process `BCFTOOLS_CONSENSUS (220603)` terminated with an error exit status (1) -- Error is ignored
    [79/3b0758] NOTE: Process `VARSCAN2_CONSENSUS (220600)` terminated with an error exit status (1) -- Error is ignored
    [00/917a76] NOTE: Process `VARSCAN2_CONSENSUS (220603)` terminated with an error exit status (1) -- Error is ignored
```

Important to check both the mapping and map.

e.g :

```
    cat 20220209_mag01.log
```
Usually there are two types of errors found:

1. **Consensus**: There are samples with empty .vcf files.

In this case the samples with this issue are 220603 and 220600

e.g:
```Bash
    [00/917a76] NOTE: Process `VARSCAN2_CONSENSUS (220603)`
```

Debug the problem:

e.g:
```
    cd work/00/917a76366f84277f5fd20150c81eb8/
    ls -l -a 

    total 1540
    drwxrwsr-x 1 1218 bioinfo   4096 feb  9 17:20 .
    drwxrwsr-x 1 1218 bioinfo   4096 feb  9 17:15 ..
    -rw-rw-r-- 1 1218 bioinfo  30410 feb  9 17:19 220603.AF0.75.consensus.masked.fa
    -rw-rw-r-- 1 1218 bioinfo 755262 feb  9 17:19 220603.AF0.75.lowcov.bed
    -rw-rw-r-- 1 1218 bioinfo 695456 feb  9 17:19 220603.AF0.75.lowcov.fix.bed
    -rw-rw-r-- 1 1218 bioinfo     20 feb  9 17:19 220603.AF0.75.mask.bed
    lrwxrwxrwx 1 1218 bioinfo    193 feb  9 17:19 220603.AF0.75.vcf.gz -> /data/bi/services_and_colaborations/CNM/virologia/SRVCNM572_20220209_SARSCOV278_icasas_S/ANALYSIS/20220209_ANALYSIS01_AMPLICONS_HUMAN/work/53/ccb29a2b7e347fb17910cf86578271/220603.AF0.75.vcf.gz
    lrwxrwxrwx 1 1218 bioinfo    197 feb  9 17:19 220603.AF0.75.vcf.gz.tbi -> /data/bi/services_and_colaborations/CNM/virologia/SRVCNM572_20220209_SARSCOV278_icasas_S/ANALYSIS/20220209_ANALYSIS01_AMPLICONS_HUMAN/work/53/ccb29a2b7e347fb17910cf86578271/220603.AF0.75.vcf.gz.tbi
    lrwxrwxrwx 1 1218 bioinfo    192 feb  9 17:19 220603.trim.mpileup -> /data/bi/services_and_colaborations/CNM/virologia/SRVCNM572_20220209_SARSCOV278_icasas_S/ANALYSIS/20220209_ANALYSIS01_AMPLICONS_HUMAN/work/ee/45f84036e4e1e6a65fd7f0502857ed/220603.trim.mpileup
    -rw-rw-r-- 1 1218 bioinfo      0 feb  9 17:19 .command.begin
    -rw-rw-r-- 1 1218 bioinfo   2148 feb  9 17:20 .command.err
    -rw-r--r-- 1 1218 bioinfo   2148 feb  9 17:20 .command.log
    -rw-rw-r-- 1 1218 bioinfo      0 feb  9 17:19 .command.out
    -rw-rw-r-- 1 1218 bioinfo  11731 feb  9 17:17 .command.run
    -rw-rw-r-- 1 1218 bioinfo    770 feb  9 17:17 .command.sh
    -rw-rw-r-- 1 1218 bioinfo      0 feb  9 17:19 .command.trace
    -rw-rw-r-- 1 1218 bioinfo      1 feb  9 17:20 .exitcode
    lrwxrwxrwx 1 1218 bioinfo     60 feb  9 17:19 NC_045512.2.fasta -> /data/bi/references/virus/2019-nCoV/genome/NC_045512.2.fasta
    -rw-rw-r-- 1 1218 bioinfo  30290 feb  9 17:19 NC_045512.2.ref.masked.fa
```

Open the file *.command.err* in the folder referenced by the log as [** /******]:

For the case [00/917a76] NOTE: Process `VARSCAN2_CONSENSUS (220603)`

```
    cat work/00/917a76366f84277f5fd20150c81eb8/.command.err 
```

e.g :
```
    Note: the --sample option not given, applying all records regardless of the genotype
    Warning: Sequence "NC_045512.2" not in 220603.AF0.75.vcf.gz
    Warning message:
    package ‘optparse’ was built under R version 3.6.3 
    Warning message:
    package ‘ggplot2’ was built under R version 3.6.3 
    Warning message:
    package ‘scales’ was built under R version 3.6.3 
    Warning message:
    package ‘reshape2’ was built under R version 3.6.3 
    Loading required package: BiocGenerics
    Loading required package: parallel

    Attaching package: ‘BiocGenerics’

    The following objects are masked from ‘package:parallel’:

        clusterApply, clusterApplyLB, clusterCall, clusterEvalQ,
        clusterExport, clusterMap, parApply, parCapply, parLapply,
        parLapplyLB, parRapply, parSapply, parSapplyLB

    The following objects are masked from ‘package:stats’:

        IQR, mad, sd, var, xtabs

    The following objects are masked from ‘package:base’:

        anyDuplicated, append, as.data.frame, basename, cbind, colnames,
        dirname, do.call, duplicated, eval, evalq, Filter, Find, get, grep,
        grepl, intersect, is.unsorted, lapply, Map, mapply, match, mget,
        order, paste, pmax, pmax.int, pmin, pmin.int, Position, rank,
        rbind, Reduce, rownames, sapply, setdiff, sort, table, tapply,
        union, unique, unsplit, which, which.max, which.min

    Loading required package: S4Vectors
    Loading required package: stats4

    Attaching package: ‘S4Vectors’

    The following object is masked from ‘package:base’:

        expand.grid

    Loading required package: IRanges
    Loading required package: XVector

    Attaching package: ‘Biostrings’

    The following object is masked from ‘package:base’:

        strsplit

    Error in 100 * base_tab$freq : non-numeric argument to binary operator
    In addition: Warning messages:
    1: In `[<-.factor`(`*tmp*`, ri, value = "A") :
    invalid factor level, NA generated
    2: In `[<-.factor`(`*tmp*`, ri, value = "C") :
    invalid factor level, NA generated
    3: In `[<-.factor`(`*tmp*`, ri, value = "T") :
    invalid factor level, NA generated
    4: In `[<-.factor`(`*tmp*`, ri, value = "G") :
    invalid factor level, NA generated
    Execution halted
```

The error *Execution halted* shows that there is a mismatch between  the reference and the .vcf file and the nucleotide is not found. This happens in deletions and insertions cases. This example is for the sample *220603*, it can be checked that is empty by doing:

```
e.g:

    zcat 220603.AF0.75.vcf.gz 
    ##fileformat=VCFv4.1
    ##FILTER=<ID=PASS,Description="All filters passed">
    ##source=VarScan2
    ##INFO=<ID=ADP,Number=1,Type=Integer,Description="Average per-sample depth of bases with Phred score >= 20">
    ##INFO=<ID=WT,Number=1,Type=Integer,Description="Number of samples called reference (wild-type)">
    ##INFO=<ID=HET,Number=1,Type=Integer,Description="Number of samples called heterozygous-variant">
    ##INFO=<ID=HOM,Number=1,Type=Integer,Description="Number of samples called homozygous-variant">
    ##INFO=<ID=NC,Number=1,Type=Integer,Description="Number of samples not called">
    ##FILTER=<ID=str10,Description="Less than 10% or more than 90% of variant supporting reads on one strand">
    ##FILTER=<ID=indelError,Description="Likely artifact due to indel reads at this position">
    ##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
    ##FORMAT=<ID=GQ,Number=1,Type=Integer,Description="Genotype Quality">
    ##FORMAT=<ID=SDP,Number=1,Type=Integer,Description="Raw Read Depth as reported by SAMtools">
    ##FORMAT=<ID=DP,Number=1,Type=Integer,Description="Quality Read Depth of bases with Phred score >= 20">
    ##FORMAT=<ID=RD,Number=1,Type=Integer,Description="Depth of reference-supporting bases (reads1)">
    ##FORMAT=<ID=AD,Number=1,Type=Integer,Description="Depth of variant-supporting bases (reads2)">
    ##FORMAT=<ID=FREQ,Number=1,Type=String,Description="Variant allele frequency">
    ##FORMAT=<ID=PVAL,Number=1,Type=String,Description="P-value from Fisher's Exact Test">
    ##FORMAT=<ID=RBQ,Number=1,Type=Integer,Description="Average quality of reference-supporting bases (qual1)">
    ##FORMAT=<ID=ABQ,Number=1,Type=Integer,Description="Average quality of variant-supporting bases (qual2)">
    ##FORMAT=<ID=RDF,Number=1,Type=Integer,Description="Depth of reference-supporting bases on forward strand (reads1plus)">
    ##FORMAT=<ID=RDR,Number=1,Type=Integer,Description="Depth of reference-supporting bases on reverse strand (reads1minus)">
    ##FORMAT=<ID=ADF,Number=1,Type=Integer,Description="Depth of variant-supporting bases on forward strand (reads2plus)">
    ##FORMAT=<ID=ADR,Number=1,Type=Integer,Description="Depth of variant-supporting bases on reverse strand (reads2minus)">
    ##bcftools_filterVersion=1.9+htslib-1.9
    ##bcftools_filterCommand=filter -i 'FORMAT/AD / (FORMAT/AD + FORMAT/RD) >= 0.75' --output-type z --output 220603.AF0.75.vcf.gz 220603.vcf.gz; Date=Wed Feb  9 17:08:44 2022
    #CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  220603
```

This .vcf contains no variants as expected.

SOLUTION: There were no variants found for those samples. This error requires that we check the vcf of the sample and see that it is empty as expected. That is it.

1. **Core dumped**: The RAM memory from HPc it is full.

e.g:

```
    [61/30f1aa] NOTE: Process `CUTADAPT (220648)` terminated with an error exit status (134) -- Error is ignored
    [83/4ebcf1] NOTE: Process `CUTADAPT (220649)` terminated with an error exit status (134) -- Error is ignored
```

As before go to *.command.err* in /work/61/30f1aa ...

```
    cat .command.err
    /data/bi/services_and_colaborations/CNM/virologia/SRVCNM572_20220209_SARSCOV278_icasas_S/ANALYSIS/20220209_ANALYSIS01_AMPLICONS_HUMAN/work/61/30f1aab151fb7c3afc530b10dbdf1e/.command.sh: line 14:  
    
    6758 Aborted                 (core dumped) fastqc --quiet --threads 5 *.ptrim.fastq.gz
```

In this case the error that was printed in the log as `CUTADAPT`, however when checking the error in work/... it is shown as a error related *core dumped*. The processes as `CUTADAPT` which use Java tend to produce this error.

*SOLUTION:* This error is solved by running again nextflow. Since nextflow is run using *-resume* the cache will run again the processed that were not finished. Before running it again, the availability of the HPC should be checked as explained before, in order to use the nodes that are not being used.

Run again the process based on the lablog.

```
    conda activate nextflow 
    nohup bash _01_viralrecon_mapping.sh &> $(date '+%Y%m%d')_mapping01.log &
```

To check the status of the processes:
```
    tail -f 20220210_mapping01.log 
    [36/90360e] Cached process > VARSCAN2_QUAST
    [50/2fc490] Submitted process > KRAKEN2 (kraken2_human)
    [c2/c28771] Submitted process > KRAKEN2 (kraken2_human)
    [9d/3b76f2] Submitted process > MULTIQC (1)
```

The processes stated as "submitted" are running again.
