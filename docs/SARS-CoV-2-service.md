## Viralrecon SARS CoV 2 service description:

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

    cd 20220209_ANALYSIS01_AMPLICONS_HUMAN/
    cat 20220209_mapping01.log 




To find the errors in the log do: 

e.g : 

    grep 'termin' 20220209_mapping01.log    
    [61/30f1aa] NOTE: Process `CUTADAPT (220648)` terminated with an error exit status (134) -- Error is ignored
    [83/4ebcf1] NOTE: Process `CUTADAPT (220649)` terminated with an error exit status (134) -- Error is ignored
    [39/1091c2] NOTE: Process `IVAR_CONSENSUS (220600)` terminated with an error exit status (1) -- Error is ignored
    [5e/0fd305] NOTE: Process `BCFTOOLS_CONSENSUS (220600)` terminated with an error exit status (1) -- Error is ignored
    [65/2c4848] NOTE: Process `IVAR_CONSENSUS (220603)` terminated with an error exit status (1) -- Error is ignored
    [f9/b2e511] NOTE: Process `BCFTOOLS_CONSENSUS (220603)` terminated with an error exit status (1) -- Error is ignored
    [79/3b0758] NOTE: Process `VARSCAN2_CONSENSUS (220600)` terminated with an error exit status (1) -- Error is ignored
    [00/917a76] NOTE: Process `VARSCAN2_CONSENSUS (220603)` terminated with an error exit status (1) -- Error is ignored

Important to check both the mapping and map.

e.g : 

    cat 20220209_mag01.log

Usually there are two types of errors found: 

1. **Consensus**: There are samples with empty .vcf files. 

In this case the samples with this issue are 220603 and 220600

e.g: 

    [00/917a76] NOTE: Process `VARSCAN2_CONSENSUS (220603)`

Debug the problem:

e.g: 
    
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

Open the file *.command.err* in the folder referenced by the log as [** /******]:

For the case [00/917a76] NOTE: Process `VARSCAN2_CONSENSUS (220603)`

    cat work/00/917a76366f84277f5fd20150c81eb8/.command.err 

e.g :

    
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

The error *Execution halted* shows that there is a mismatch between  the reference and the .vcf file and the nucleotide is not found. This happens in deletions and insertions cases. This example is for the sample *220603*, it can be checked that is empty by doing: 

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

This .vcf contains no variants as expected. 

SOLUTION: There were no variants found for those samples. This error requires that we check the vcf of the sample and see that it is empty as expected. That is it. 

1. **Core dumped**: The RAM memory from HPc it is full. 

e.g: 

    [61/30f1aa] NOTE: Process `CUTADAPT (220648)` terminated with an error exit status (134) -- Error is ignored
    [83/4ebcf1] NOTE: Process `CUTADAPT (220649)` terminated with an error exit status (134) -- Error is ignored


As before go to *.command.err* in /work/61/30f1aa ... 

    cat .command.err
    /data/bi/services_and_colaborations/CNM/virologia/SRVCNM572_20220209_SARSCOV278_icasas_S/ANALYSIS/20220209_ANALYSIS01_AMPLICONS_HUMAN/work/61/30f1aab151fb7c3afc530b10dbdf1e/.command.sh: line 14:  
    
    6758 Aborted                 (core dumped) fastqc --quiet --threads 5 *.ptrim.fastq.gz



In this case the error that was printed in the log as `CUTADAPT`, however when checking the error in work/... it is shown as a error related *core dumped*. The processes as `CUTADAPT` which use Java tend to produce this error. 

SOLUTION: This error is solved by running again nextflow. Since nextflow is run using *-resume* the cache will run again the processed that were not finished. Before running it again, the availability of the HPC should be checked as explained before, in order to use the nodes that are not being used. 

Run again the process based on the lablog. 

    conda activate nextflow 
    nohup bash _01_viralrecon_mapping.sh &> $(date '+%Y%m%d')_mapping01.log &

To check the status of the processes: 

    tail -f 20220210_mapping01.log 
    [36/90360e] Cached process > VARSCAN2_QUAST
    [50/2fc490] Submitted process > KRAKEN2 (kraken2_human)
    [c2/c28771] Submitted process > KRAKEN2 (kraken2_human)
    [9d/3b76f2] Submitted process > MULTIQC (1)

The processes stated as "submitted" are running again. 


2. PANGOLING linages


The next step is to obtain the linaged from the consensus given a multifasta as input for Pangolin software. Pangolin must be updated in advanced before executing this step. 

    cd 20220209_pangolin/
    conda activate 
    pangolin --update
    pangolin --update data


The lablog should look something like this: 

    cat lablog
    #conda activate pangolin
    #pangolin --update
    #pangolin --update-data
    cat ../*_viralrecon_mapping/variants/varscan2/consensus/*.consensus.masked.fa > human_masked.fasta
    echo "qsub -V -b y -j y -cwd -N PANGOLIN -q all.q pangolin human_masked.fasta" > _01_pangolin.sh
    pangolin -v  >> version.log
    pangolin -pv >> version.log
    pangolin -dv >> version.log
    #cat ../samples_id.txt | while read in; do grep -P "${in}," lineage_report.csv; done | tr ',' '\t' | cut -f2
    #cat ../samples_id.txt | while read in; do grep -P "${in}," lineage_report.csv; done | tr ',' '\t' | cut -f2 | sort -u | grep -v 'None' | sed ':a;N;$!ba;s/\n/, /g'


Execute the lablog 

    bash lablog 

Open *_01_pangolin.sh* and execute it. 

    cat _01_pangolin.sh 
    qsub -V -b y -j y -cwd -N PANGOLIN -q all.q pangolin human_masked.fasta

    bash _01_pangolin.sh

To save the logs the logs folder is created. Then the files are moved into the folder. 

     mkdir logs
     mv PANGOLIN.o1218624 logs/

Inside the folder the log can be checked in the following way: 

    cd logs 
    cat PANGOLIN.o1207194 
    Job stats:
    job                     count    min threads    max threads
    --------------------  -------  -------------  -------------
    add_failed_seqs             1              1              1
    align_to_reference          1              1              1
    all                         1              1              1
    generate_report             1              1              1
    get_constellations          1              1              1
    hash_sequence_assign        1              1              1
    pangolearn                  1              1              1
    scorpio                     1              1              1
    total                       8              1              1

    /data/bi/pipelines/miniconda3/envs/pangolin/lib/python3.8/site-packages/sklearn/base.py:329: UserWarning: Trying to unpickle estimator DecisionTreeClassifier from version 0.24.2 when using version 0.23.1. This might lead to breaking code or invalid results. Use at your own risk.
    warnings.warn(
    loading model 01/27/2022, 23:26:00
    processing block of 81 sequences 01/27/2022, 23:26:02
    complete 01/27/2022, 23:26:02
    Output file written to: /data/bi/services_and_colaborations/CNM/virologia/SRVCNM565_20220127_SARSCOV277_icasas_S/ANALYSIS/20220127_ANALYSIS01_AMPLICONS_HUMAN/20220127_pangolin/lineage_report.csv
    All dependencies satisfied.
    The query file is:/data/bi/services_and_colaborations/CNM/virologia/SRVCNM565_20220127_SARSCOV277_icasas_S/ANALYSIS/20220127_ANALYSIS01_AMPLICONS_HUMAN/20220127_pangolin/human_masked.fasta
    ** Running sequence QC **
    Number of sequences detected: 95
    Total passing QC: 81

    Data files found:
    Trained model:  /data/bi/pipelines/miniconda3/envs/pangolin/lib/python3.8/site-packages/pangoLEARN/data/decisionTree_v1.joblib
    Header file:    /data/bi/pipelines/miniconda3/envs/pangolin/lib/python3.8/site-packages/pangoLEARN/data/decisionTreeHeaders_v1.joblib
    Designated hash:        /data/bi/pipelines/miniconda3/envs/pangolin/lib/python3.8/site-packages/pangoLEARN/data/lineages.hash.csv


The csv file with PANGOLIN result is checked:  
    
    cat lineage_report.csv 

The last lines of the file show which samples failed. 

    220462,BA.1,0.0,0.9503078276165348,Omicron (BA.1-like),0.931000,0.017200,PLEARN-v1.2.123,3.1.18,2022-01-20,v1.2.123,passed_qc,scorpio call: Alt alleles 54; Ref alleles 1; Amb alleles 1; Oth alleles 2
    220463,AY.43,0.0,0.9700107874865156,Delta (B.1.617.2-like),1.000000,0.000000,PLEARN-v1.2.123,3.1.18,2022-01-20,v1.2.123,passed_qc,scorpio call: Alt alleles 13; Ref alleles 0; Amb alleles 0; Oth alleles 0
    220474,BA.1.1,0.0,0.9264672813132449,Omicron (BA.1-like),0.931000,0.017200,PLEARN-v1.2.123,3.1.18,2022-01-20,v1.2.123,passed_qc,scorpio call: Alt alleles 54; Ref alleles 1; Amb alleles 1; Oth alleles 2
    220356,None,,,,,,PANGO-v1.2.123,3.1.18,2022-01-20,v1.2.123,fail,N_content:0.69
    220357,None,,,,,,PANGO-v1.2.123,3.1.18,2022-01-20,v1.2.123,fail,N_content:0.37
    220358,None,,,,,,PANGO-v1.2.123,3.1.18,2022-01-20,v1.2.123,fail,N_content:0.56
    220359,None,,,,,,PANGO-v1.2.123,3.1.18,2022-01-20,v1.2.123,fail,N_content:0.33
    220392,None,,,,,,PANGO-v1.2.123,3.1.18,2022-01-20,v1.2.123,fail,N_content:0.37
    220412,None,,,,,,PANGO-v1.2.123,3.1.18,2022-01-20,v1.2.123,fail,N_content:0.57



Now we checked create the long and wide tables: 

    cd 20220209_variants_table/
    ls
    

The lablog should look like this: 

    cat lablog
    #conda activate nf-core-viralrecon-1.2.0dev
    cat ../samples_id.txt | while read in; do ln -s ../*_viralrecon_mapping/variants/varscan2/${in}.AF0.75.vcf.gz* .; done
    #remove samples that didn't finish

    VCF_FILES=$(ls *.gz | tr '\n' ' ') 
    echo "qsub -V -b y -j y -cwd -N BCFTOOLS_MERGE -q all.q bcftools merge -m none $VCF_FILES-Ov -o all_samples_merged.vcf" > _01_bcftools_merge.sh
    echo "bcftools query -H -f '%POS\t%REF\t%ALT\t[%DP\t]\n' all_samples_merged.vcf > all_variants.table" > _02_bcftools_query.sh
    echo "sed -i 's/\# //g' all_variants.table" > _03_parse_table.sh
    echo "sed -i 's/\[[0-9]*\]//g' all_variants.table" >> _03_parse_table.sh
    #Create template for SnpSift annotation
    cat ../*_viralrecon_mapping/variants/varscan2/snpeff/*.AF0.75.snpSift.table.txt | sort -u > snpSift_template.txt
    cat snpSift_template.txt | cut -f2,3,4,5,8,13,14 > snpSift_template_filtered.txt
    cat ../*_pangolin/lineage_report.csv | tr ',' '\t' | cut -f 1,2 | sort -u | head -n -1 > samples_lineage.txt
    echo "qsub -V -b y -j y -cwd -N CREATE.VARIANTS.TABLE -q all.q Rscript parser.R" > _04_create_tables.sh
    cp /data/bi/services_and_colaborations/CNM/virologia/SRVCNM349_20210308_SARSCOV236_icasas_S/ANALYSIS/20210308_ANALYSIS01_AMPLICONS_HUMAN/20210324_variants_table/parser.R .

Execute the lablog, activating the environment. 

    conda activate nf-core-viralrecon-1.2.0dev
    bash lablog 

    cat _01_bcftools_merge.sh 
    qsub -V -b y -j y -cwd -N BCFTOOLS_MERGE -q all.q bcftools merge -m none 220356.AF0.75.vcf.gz 220357.AF0.75.vcf.gz 220358.AF0.75.vcf.gz 220359.AF0.75.vcf.gz 220360.AF0.75.vcf.gz 220361.AF0.75.vcf.gz 220362.AF0.75.vcf.gz 220363.AF0.75.vcf.gz 220364.AF0.75.vcf.gz 220365.AF0.75.vcf.gz 220366.AF0.75.vcf.gz 220367.AF0.75.vcf.gz 220368.AF0.75.vcf.gz 220369.AF0.75.vcf.gz 220370.AF0.75.vcf.gz 220371.AF0.75.vcf.gz 220372.AF0.75.vcf.gz 220373.AF0.75.vcf.gz 220374.AF0.75.vcf.gz 220375.AF0.75.vcf.gz 220376.AF0.75.vcf.gz 220377.AF0.75.vcf.gz 220378.AF0.75.vcf.gz 220389.AF0.75.vcf.gz 220390.AF0.75.vcf.gz 220392.AF0.75.vcf.gz 220393.AF0.75.vcf.gz 220394.AF0.75.vcf.gz 220395.AF0.75.vcf.gz 220396.AF0.75.vcf.gz 220397.AF0.75.vcf.gz 220398.AF0.75.vcf.gz 220399.AF0.75.vcf.gz 220400.AF0.75.vcf.gz 220401.AF0.75.vcf.gz 220402.AF0.75.vcf.gz 220403.AF0.75.vcf.gz 220404.AF0.75.vcf.gz 220405.AF0.75.vcf.gz 220406.AF0.75.vcf.gz 220408.AF0.75.vcf.gz 220409.AF0.75.vcf.gz 220410.AF0.75.vcf.gz 220411.AF0.75.vcf.gz 220412.AF0.75.vcf.gz 220413.AF0.75.vcf.gz 220414.AF0.75.vcf.gz 220415.AF0.75.vcf.gz 220416.AF0.75.vcf.gz 220417.AF0.75.vcf.gz 220418.AF0.75.vcf.gz 220419.AF0.75.vcf.gz 220420.AF0.75.vcf.gz 220421.AF0.75.vcf.gz 220422.AF0.75.vcf.gz 220423.AF0.75.vcf.gz 220424.AF0.75.vcf.gz 220425.AF0.75.vcf.gz 220426.AF0.75.vcf.gz 220427.AF0.75.vcf.gz 220428.AF0.75.vcf.gz 220429.AF0.75.vcf.gz 220430.AF0.75.vcf.gz 220431.AF0.75.vcf.gz 220432.AF0.75.vcf.gz 220434.AF0.75.vcf.gz 220435.AF0.75.vcf.gz 220436.AF0.75.vcf.gz 220437.AF0.75.vcf.gz 220438.AF0.75.vcf.gz 220439.AF0.75.vcf.gz 220440.AF0.75.vcf.gz 220441.AF0.75.vcf.gz 220442.AF0.75.vcf.gz 220443.AF0.75.vcf.gz 220444.AF0.75.vcf.gz 220445.AF0.75.vcf.gz 220446.AF0.75.vcf.gz 220447.AF0.75.vcf.gz 220448.AF0.75.vcf.gz 220449.AF0.75.vcf.gz 220451.AF0.75.vcf.gz 220452.AF0.75.vcf.gz 220453.AF0.75.vcf.gz 220454.AF0.75.vcf.gz 220455.AF0.75.vcf.gz 220456.AF0.75.vcf.gz 220457.AF0.75.vcf.gz 220458.AF0.75.vcf.gz 220459.AF0.75.vcf.gz 220460.AF0.75.vcf.gz 220461.AF0.75.vcf.gz 220462.AF0.75.vcf.gz 220463.AF0.75.vcf.gz 220474.AF0.75.vcf.gz -Ov -o all_samples_merged.vcf

    bash _01_bcftools_merge.sh

Again create a logs folder and move the logs. 

    mkdir logs
    mv BCFTOOLS_MERGE.o1218625 logs/

Keep executing the .sh files in order (1,2,3 etc): 

    cat _02_bcftools_query.sh 
    bcftools query -H -f '%POS\t%REF\t%ALT\t[%DP\t]\n' all_samples_merged.vcf > all_variants.table

    bash _02_bcftools_query.sh

    cat _03_parse_table.sh 
    sed -i 's/\# //g' all_variants.table
    sed -i 's/\[[0-9]*\]//g' all_variants.table

    bash _03_parse_table.sh

    cat _04_create_tables.sh 
    qsub -V -b y -j y -cwd -N CREATE.VARIANTS.TABLE -q all.q Rscript parser.R

    bash _04_create_tables.sh


Move the logs again into de logs folder and check it: 

    mv CREATE.VARIANTS.TABLE.o1218626 logs/

    cat CREATE.VARIANTS.TABLE.o1218626
    Warning message:
    package ‘reshape2’ was built under R version 3.6.3 



When the tables (Long and Wide) are created then we save them into **.xlsx** format and at this step we also check that the table is created correctly and go through the file checking for special variants. 

To check the file we look into the REF  and ALT columns. Insertions and deletions are usually the rare variants to take into account. 

If there are variants that raise any doubt then IGV is used to look into the specific position to check that the Allele frequency, the Depth are correct. 


In this case there was an insertion at position 22204. To check it, go to the vcf file and software, varscan2 in this case. 

    cd 20220209_viralrecon_mapping/variants/varscan2/

    zcat *.AF0.75.vcf.gz | grep '22204'
    NC_045512.2     22204   .       T       TGAGCCAGAA      .       PASS    ADP=16;WT=0;HET=0;HOM=1;NC=0    GT:GQ:SDP:DP:RD:AD:FREQ:PVAL:RBQ:ABQ:RDF:RDR:ADF:ADR    1/1:0:16:16:3:13:81.25%:9.8E-1:38:38:3:0:6:7
    NC_045512.2     22204   .       T       TGAGCCAGAA      .       PASS    ADP=16;WT=0;HET=0;HOM=1;NC=0    GT:GQ:SDP:DP:RD:AD:FREQ:PVAL:RBQ:ABQ:RDF:RDR:ADF:ADR    1/1:0:16:16:3:13:81.25%:9.8E-1:38:38:1:2:5:8
    NC_045512.2     22204   .       T       TGAGCCAGAA      .       PASS    ADP=11;WT=0;HET=0;HOM=1;NC=0    GT:GQ:SDP:DP:RD:AD:FREQ:PVAL:RBQ:ABQ:RDF:RDR:ADF:ADR    1/1:0:11:11:2:9:81.82%:9.8E-1:37:38:2:0:4:5


Now the mag log is checked for errors. 

    cd 20220209_ANALYSIS02_MET/
    cat 20220209_mag01.log

    grep 'termin' 20220209_mag01.log 
    [1e/c42004] NOTE: Process `get_software_versions` terminated with an error exit status (1) -- Error is ignored

This error is normal due to software version and will be shown always due to configuration

Since there was an error with MULTIQC and had to be executed again, now that the process has finished the statistics table is created. This must be done after pangolin and variants. 

The output from MULTIQC is placed in this location

    cd 20220209_viralrecon_mapping/multiqc

    ls
    multiqc_data  multiqc_report.html


At this location */data/bi/services_and_colaborations/CNM/virologia/SRVCNM565_20220127_SARSCOV277_icasas_S/ANALYSIS/20220127_ANALYSIS01_AMPLICONS_HUMAN* we excute *_03_create_summary.sh* 

    cat _03_create_summary.sh 
    bash create_summary_report.sh ./20220127_viralrecon_mapping/variants/summary_variants_metrics_mqc.tsv ./20220127_viralrecon_mapping/variants/varscan2/consensus/ ./20220127_viralrecon_mapping/assembly/kraken2/ ./samples_id.txt summary_report.tab 'MiSeq_GEN_265_ICasas\ticasas\thuman\tNC_045512.2'

    bash _03_create_summary.sh

Now the *summary_report.tab* that was previously created is checked (Showing here a few lines) 

    less summary_report.tab
    MiSeq_GEN_265_ICasas    icasas  human   NC_045512.2     220376  273412  7510    15020   5,49    255996  93,63   2396    0,88    678,17  83,98   59      40      12,81   BA.1
    MiSeq_GEN_265_ICasas    icasas  human   NC_045512.2     220377  275084  5182    10364   3,77    261907  95,21   2813    1,02    663,29  79,98   49      34      17,77   BA.1
    MiSeq_GEN_265_ICasas    icasas  human   NC_045512.2     220378  230262  19275   38550   16,74   186627  81,05   5085    2,21    447,17  71,86   42      30      23,03   BA.1
    MiSeq_GEN_265_ICasas    icasas  human   NC_045512.2     220389  278740  15135   30270   10,86   244037  87,55   4433    1,59    597,14  75,70   42      30      20,98   BA.1.1
    MiSeq_GEN_265_ICasas    icasas  human   NC_045512.2     220390  251890  10761   21522   8,54    227381  90,27   2987    1,19    566,19  78,83   46      32      19,47   BA.1
    MiSeq_GEN_265_ICasas    icasas  human   NC_045512.2     220392  294920  31804   63608   21,57   179459  60,85   51853   17,58   332,69  56,11   33      22      36,65   None
    MiSeq_GEN_265_ICasas    icasas  human   NC_045512.2     220393  251576  4504    9008    3,58    239022  95,01   3546    1,41    580,99  75,60   47      33      21,11   BA.1
 


