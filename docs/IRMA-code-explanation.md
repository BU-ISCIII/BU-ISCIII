# IRMA

[IRMA](https://wonder.cdc.gov/amd/flu/irma/) was designed for the robust assembly, variant calling, and phasing of highly variable RNA viruses. Currently IRMA is deployed with modules for influenza, ebolavirus and coronavirus. IRMA is free to use and parallelizes computations for both cluster computing and single computer multi-core setups. For more info, read the [manuscript](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-016-3030-6)

## Command line

```bash
flu-amd/IRMA FLU_AD SAMPLE-NAME_S5_R1_001.fastq.gz SAMPLE-NAME_S5_R2_001.fastq.gz results/SAMPLE-NAME --external-config custom_config.sh
```

Command line must-have order:

1. Module. Eg: FLU_AD
2. R1 R2. Eg: SAMPLE-NAME_S5_R1_001.fastq.gz SAMPLE-NAME_S5_R2_001.fastq.gz
3. Run. Eg: results/SAMPLE-NAME (This can be a folder or a path).
4. --external-config. Custom config file.

## Some important variables

- `$bpath`: Path to where the executable with its resources and the LABEL is located (this should not be changed)
- `owd`: Output working directory. Created as `pwd` in the code
- `mpath`: Path to where the modules are located. **Possibility to customize this variable for RSV**.
- `ppath`: Path to the "work" folder: `flu-amd/IRMA_RES/ppath/{sample_name}-{unique_hash}/`

## Order in which configurations are loaded (source):

1. `default.sh`: The one inside `IRMA_RES`.
2. `init.sh`: The one inside the module being used (RSV, FLU_AD).
3. _Module config_: The one inside the config folder (minion, avian...).
4. _External config_: The one passed as a parameter (loaded with a .pl script, which first processes it to check that it's correct and then sources it).

## Steps of the analysis in `IRMA`

After checking dependencies, creating paths with the variables, setting options for each software based on the config parameters, etc., the pipeline starts around line 1672 of the `IRMA` code.

### 0. Pre-processing

1. _Disk check_: Checks if there is enough space for the analysis. This step can be skipped with the parameter `ALLOW_DISK_CHECK` set to 0.
2. _FastQconverter_: It is a custom Perl script. It handles quality and size filtering, trimming, and adapter masking.

    - **Possible parameters**:
      - `-T, --read-quality <threshold>`: Specify the read quality threshold (geometric mean or median).
      - `-M, --use-median`: Interprets the threshold (-T) as the median, not the average.
      - `-Q, --fastQ-output`: Outputs in fastQ format instead of fasta.
      - `-L, --min-length <threshold>`: Minimum length of sequence data, default = 0.
      - `-C, --complement-add`: Take the reverse complement and add to the data.
      - `-O, --ordinal-headers`: Replace header with strict ordinals.
      - `-F, --file-id <STR>`: File ID for ordinals.
      - `-S, --save-quality <STR>`: Save quality file for back-mapping.
      - `-A, --save-stats <STR>`: Save quality vs. length statistics file for analysis.
      - `-K, --skip-remaining`: Do not output FASTA/FASTQ data (assumes -A).
      - `-H, --keep-header`: Keep header as usual.
      - `-c, --clip-adapter <STR>`: Clip adapter.
      - `-m, --mask-adapter <STR>`: Mask adapter.
      - `-Z, --fuzzy-adapter`: Allow one mismatch.
      - `-U, --uracil-to-thymine`: Convert uracil to thymine.
      - `-E, --enforce-clipped-length`: The minimum length threshold (-L) is enforced when adapter clipped (-c).
      - `-R, --read-side <INT>`: If FASTQ header is in SRA format and missing a read identifier, alter the header.

    - It uses **options** from the configuration at this step such as:
      - `ADAPTER`: Transposase adapter sequence. To disable, set it as an empty string (ADAPTER=""). It trims 5′ on the forward adapter and 3′ on the reverse complement adapter. It can be applied to NextTera paired-end reads.
      - `FUZZY_ADAPTER`: If ADAPTER is set and FUZZY_ADAPTER is enabled, it also trims adapters with up to 1 mismatch.
      - `ENFORCE_CLIPPED_LENGTH`: If ADAPTER is set and ENFORCE_CLIPPED_LENGTH is enabled, the MIN_LEN will be applied to adapter-trimmed reads.
      - `QUAL_THRESHOLD`: Minimum threshold to filter reads by mean or average quality.
      - `MIN_LEN`: Minimum read length to include reads in the read collection. This value should not be greater than the typical read length.
      - `USE_MEDIAN`: Use median read quality to filter reads instead of average read quality.

    - **Default parameters**: `-U -Q -T 30 -L 125 --keep-header -M -c AGATGTGTATAAGAGACAG -Z`

    - The **output** is used as input for step 3.

3. _xflate_: Custom Perl script. It performs two main operations depending on the options provided:
- Deflation (`-C`, `--cluster-all`): Groups identical sequences into clusters and generates a table file describing these clusters.
- Inflation (`-I`, `--inflate`): Uses the table file to reconstruct the original sequences.
    In this third step, it clusters the reads and assigns them an identifier (this identifier is hardcoded in the code as `INTER`)

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/xflate.pl -C -Q -L INTER flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/R0.xfl <(zcat < SAMPLE-NAME_S5_R1_001.fastq.gz | flu-amd/IRMA_RES/scripts/parallel --nn --pipe -L4 -j1 -q flu-amd/IRMA_RES/scripts/fastQ_converter.pl  -U -Q -T 30 -L 125 --keep-header -M -c AGATGTGTATAAGAGACAG -Z -R 1 -G flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/LEFT.log -g {#}) <(zcat < SAMPLE-NAME_S5_R2_001.fastq.gz | flu-amd/IRMA_RES/scripts/parallel --nn --pipe -L4 -j1 -q flu-amd/IRMA_RES/scripts/fastQ_converter.pl  -U -Q -T 30 -L 125 --keep-header -M -c AGATGTGTATAAGAGACAG -Z -R 2 -G flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/RIGHT.log -g {#})
      ```

    - **Parameters**
        - `-C or --cluster-all`: Groups all sequences into clusters, even if they are not identical.
        - `-L or --file-label`: Adds a label to the clusters.
        - `-Q or --fastQ`: Specifies that input is expected in FASTQ format and output in the same format.

    - **Output**
      - `R0.fa`

        ```bash
        >C0%5%|INTER
        AAACAGAATGGTGCTGGCTAGCACTACGGCAAAGGCTATGGAACAGGTGGCTGGATCGAGTGAACAGGCAGCGGAGGCCATGGAGGTTGCTAATAAGACTAGGCAGATGGTACATGCAATGAGAACTATTGGAACTCATCCTAGCTC
        >C1%1%|INTER
        AGATTGAATAGACGGGACATTCCTCAATCCTGTGGCCAGTCTCAATTTTGTGCTTCTTACATACTTTGGACATTTCCCAATCGTGATCGGATGTACATTTTGAAATGGGGGGCTGGTGTTTATAGCACCCTCGGG
        ```

      - `R0.xfl`

        ```bash
        C0%5%|INTER	@M03352:453:000000000-LBGBK:1:1102:25588:19493 1:N:0:GACGAGATTA+ACATTATCCT	CCCCCFFFFFFFGGGGGGGGGGHHHHHGGGGGHHHHHHHHHHHHHHHGHGHHGGHHHGGGHHHHHHHHGHGGGGGGGGGHHHHFHGFHHHHHHHHHHHHHHHHHHGGHHHHHHHHHHHHHHGHHHHHGHHHHHHHHHHHHHHHHHHH	@M03352:453:000000000-LBGBK:1:1107:10240:26654 1:N:0:GACGAGATTA+ACATTATCCT	BCCBAFFFFFFBFGGGGEFBGFFGFHHFGGGGHHHHHFHHGGGGFFGBFEGGFEGDGEFGBFFHHHHHEGGEGGGGG?FGGGFGHG?FGFGHHFGFHFG?FFGFGEGGEGBFGGGFGGHGHFGHFFDGFFGHHHFHFGDGHHGHGFH	@M03352:453:000000000-LBGBK:1:2105:27574:17757 1:N:0:GACGAGATTA+ACATTATCCT	CCCCCFFFFFFFGGGGGGGGGGHHHHHGGGGGHHHHHHHHHHHHHHHHHGHHGGHHHGGGHHHHHHHHGHGGGGGGGGGHHHHHHGEHHHHHHHHHHHHHHHHHHGGHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH	@M03352:453:000000000-LBGBK:1:2107:23799:21342 2:N:0:GACGAGATTA+ACATTATCCT	CCCCCFFFFFFFGGGGGGGGGGHHHHHGGGGGHHHHHHHHHHHHHHGGHGHHEEHHHGGGHHGGHHGHGGGGGGEGGGGGHFHGHGEGFHHHHGHHHGHHHHHHHGGHHHGHHGHHHHGHHHHHHHHHHHHHHHHHHHHHHHHGFHF	@M03352:453:000000000-LBGBK:1:2111:17098:23426 2:N:0:GACGAGATTA+ACATTATCCT	CDCDCFFFFFFFGGGGGGGGGGHHHHHGGGGGHHHHHHHHHHHHHHHFHGHHGGHHHGGGHHHHHHHHGHGGGGGGGGGGHHHHHGAGHHHHHHHHHHHHHHHHHGGHHHGHHHHHHHHHHGHHHGHHHHHGGHHHHHFHHHHHHHH
        C1%1%|INTER	@M03352:453:000000000-LBGBK:1:1103:11151:16943 1:N:0:GACGAGATTA+ACATTATCCT	BCBCCFFFFFFFGGGGGGGGGGHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHGHHHHHGGGGGHHHHHHHHHGHHHHHHGGGGGGGGGGGHHHHBGHHFGHHGGG
        ```

### 1. Do round (Repeated until $MAX_ROUNDS)

#### 1.1. Do MATCH

4. _interleavedSamples_: Custom Perl script. It distributes the fasta output from _xflate_ (3) into groups based on the number of processes (cpus) specified.

    - Uses **options** from the configuration at this step such as:
      - `SINGLE_LOCAL_PROC`: For task parallelization on the computer running IRMA, typically the number of logical CPU cores.
      - `MATCH_PROC`: If grid execution is enabled and the quantity limit is exceeded, number of array tasks for the match step.

    - **Possible parameters**:
      - `-G, --groups`: Number of groups into which sequences will be divided.
      - `-F, --fraction`: Fraction of the dataset (cannot be used together with --groups).
      - `-Q, --fastQ`: Indicates that the input and output files are in FASTQ format.
      - `-P, --by-read-pairs`: Uses FASTQ format and interleaves by read pairs based on molecular ID.
      - `-Z, --read-zipped`: Indicates that the input file is compressed.
      - `-U, --underscore-header`: Replaces spaces in headers with underscores.
      - `-R, --remove-empty-files`: Removes empty output files.
      - `-S, --silence`: Silences the summary output.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/interleavedSamples.pl --silence -G 2 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/R0.fa flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/MATCH/R1-all
      ```

    - **Output**:
      - If we remove the --silence flag, it will tell us this (in this example I told it to use only two cores)

        ```bash
        Total   Got Sample Name
        ----------------------------------------------------
        72100   36050   flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hasht/MATCH/R1-all_0001.fasta
        72100   36050   flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hasht/MATCH/R1-all_0002.fasta
        ----------------------------------------------------
        ```

      - The `-all_00X.fasta`: Which are like the original:

        ```bash
        >C0%3%|INTER
        atccagcttctttttccattgtgaatgcatacctcggtgccactagattaccagttgcttcgaatgttattttgtctcccggttctactagtgtccagtaatagttcattctccctgcttgatccctcactttgggtcttgctgctattt
        >C2%1%|INTER
        ctgcctgttcactcgatccagccacccgttccatagcctttgccgtagtgctagccagcaccattctgttttcatgcctgattaatggatttgtggtagtagccatttgtctgtgagaccgatgctgtgaatcagcaatctgttcacaagt
        ```

#### 1.2 doBLAT

5. _BLAT_: Uses the output from _interleavedSamples_ (4) and the reference (`flu-amd/IRMA_RES/modules/FLU_AD/reference/consensus.fasta` in this example) to perform BLAT. The [BLAT](https://genome.ucsc.edu/FAQ/FAQblat.html#blat1) (BLAST-Like Alignment Tool) is especially efficient for aligning long sequences against large reference databases. BLAT uses a heuristic algorithm that prioritizes speed over sensitivity. While it is less sensitive than some exhaustive alignment methods like BLAST, it generally produces accurate and reliable results.

    - Uses (may use) **options** from the configuration at this step such as:
      - `MATCH_PROG`: 
      - `MIN_BLAT_MATCH`: Minimum BLAT match length to include reads during the read collection phase. The default settings are effectively 30bp even when set to a lower value; setting to 0 disables length checking. Final match states in the assembly may be shorter due to SSW or MINIMAP2 trimming. This value should not be greater than the typical read length.
      - `LIMIT_BLAT`: If grid execution is enabled, uses the scheduler for BLAT when available read patterns per sample are ≥ the threshold.
      - `GENE_GROUP`: Necessary for two-stage sorting. The first stage uses BLAT to sort into approximate groups that can be further sorted using SECONDARY_LABEL_MODULES. The format is a comma-separated list of patterns with an optional alias at the end for gene names not found in the list ("otherwise"). Example: HA,NA:OG
      - `INCL_CHIM`: Include chimeric reads found. Chimeric reads contain both strands.
      - `SKIP_E`: Skip reference elongation/extension at the 5′ and 3′ ends. Used during the read collection phase.
      - `MIN_BLAT_MATCH`: Minimum BLAT match length to include reads during the read collection phase. The default setting is effectively 30 bp even when set to a lower value; setting to 0 disables length checking. Final match states in the assembly may be shorter due to SSW or MINIMAP2 trimming. This value should not be greater than the typical read length.
      - `SORT_PROG`: Program for the sorting step, used to determine the best match for each paired read. BLAT in this example.
      - `ALIGN_PROG`: Program for the alignment step, used to roughly align and edit each reference for the next round.
      - `SECONDARY_SORT`: Applies to the sorting LABEL. If enabled, performs a two-stage sort using SECONDARY_LABEL_MODULES after an initial sort by BLAT according to GENE_GROUP; otherwise, uses LABEL_MODULE for all matching segments. Not to be confused with secondary data.
      - `REF_SET`: Reference set used for the first round of read collection. Contained in the module's reference folder.

    - **Command line**:

      ```bash
        LANG=POSIX
        shopt -u nocaseglob;shopt -u nocasematch

        if [ "$#" -eq "1" ]; then 
          ID=$(printf %04d $1)
        else
          ID=$(printf %04d $SGE_TASK_ID)
        fi
        INPUT="flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/MATCH"/R1-all_$ID.fasta
        OUTPUT="flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/MATCH"/R1-all_$ID.blat
        # GENES/R0.ref == reference/consensus.fasta

        "flu-amd/IRMA_RES/scripts/blat_Darwin" "flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/GENES/R0.refs" "$INPUT" "$OUTPUT" -oneOff=1 -minIdentity=80 -tileSize=10
        "flu-amd/IRMA_RES/scripts"/parseBlat.pl "$OUTPUT" "$INPUT" -C -S -P "R1"
      ```

    - **Output**:
      - `R1-all_0001.blat`

        ```bash
        psLayout version 3

        match	mis- 	rep. 	N's	Q gap	Q gap	T gap	T gap	strand	Q        	Q   	Q    	Q  	T        	T   	T   T  	block	blockSizes 	qStarts	 tStarts:
          match	match	   	count	bases	count	bases	      	name     	size	start	end	name     	size	starend	count
        ---------------------------------------------------------------------------------------------------------------------------------------------------------------
        138	13	0	0	0	0	0	0	+	C0%1%|INTER	151	0	151	A_MP	982	193	344	151,	0,	193,
        124	6	0	0	0	0	0	0	-	C2%1%|INTER	130	0	130	A_PB2	2280	2052	2182130,	0,	2052,
        127	22	0	0	0	0	0	0	+	C4%1%|INTER	150	0	149	A_NS	863	626	775	149,	0,	626,
        127	9	0	0	0	0	0	0	-	C6%1%|INTER	136	0	136	A_MP	982	12	148	136,	0,	12,
        132	18	0	0	0	0	0	0	+	C8%1%|INTER	150	0	150	A_MP	982	433	583	150,	0,	433,

        ```

      - `R1-all_0001.class`

        ```bash
        C0%1%|INTER	A_MP	125
        C2%1%|INTER{c}	A_PB2	118
        C4%1%|INTER	A_NS	105
        C6%1%|INTER{c}	A_MP	118
        C8%1%|INTER	A_MP	114
        C10%1%|INTER{c}	A_NA_N1	132
        C12%1%|INTER	A_NS	104
        C14%1%|INTER	A_NP	70
        C16%1%|INTER	A_PA	131
        C20%1%|INTER	A_HA_H1	119
        ```

      - `R1-all_0001.chim`

        ```bash
        >C632%1%|INTER
        tctttcttaatccgtccagactcgactctggcatcaatccgggccctagacaccatggacacagtaaacagaacacaccaatactcagaaaaggggaaatggacaacaaacacagaaactggtgcacctcagctcaacccgatgtg
        >C4414%1%|INTER
        gtcaatgtaacattgctggctggatcttgggaaatccagagtgtgaatcactctccacagcacgaggatttctttccctttatcattaatgtaggtttggttgatctttgggtacgtttttccttttttaaccagccatatcaagtttttg
        >C4922%1%|INTER
        gatgcatatgtttttgtggggacatcaagatccagccagcaatgttacattgacccaaatgcaatggggctacccctcttagtttgcatagttttccgttatgcttgtcttctagaagattgacagagtgtgttactgttacattctttt
        ```

      - `R1-all_0001.match`

        ```bash
        >C0%1%|INTER
        cgctcaccgtgcccagtgagcgaggactgcagcgtagacgctatatccaaaatgccctaaatggaaatggggacccgaacaacatggatagagcagttagactatacaagaaactcaaaagagaaataacgttccatggggccaaagaagt
        >C2%1%|INTER{c}
        ggggtggagtctgctgtcctgagaggattcctcattttaggcaaagaagacaagaggtatggcccagcattaagcatcaatgaactgagcaatcttgctaaaggagagaaagctaatgtgctaattgggc
        >C4%1%|INTER
        tgggagaccttcactacctccagagcagaaatgagaaatggcgggaacaattgggacagaaatttgaggaaataagatgggtaattgaagaaatacgacacagattgaaagcgacagaaaatagtttcgagcaaataacatttatgcaaa
        ```

      - `R1-all_0001.nomatch`

        ```bash
        >C18%1%|INTER
        ggttcatgcttatgcctaggcaaaagataataggccctctttgcgtgcgattggaccaggcggtcatggaaaagaacatagtactggaagcaaacttcagtgtaatcttcaacc

#### 1.3. Do SORT

As the default `SORT_PROG` is the same as the `MATCH_PROG`, it assumes that this step is already done, so it would only perform parseSORTresuslts.

6. _parseSORTresults_: Perl script. Analyzes the results generated by SORT (Sequence Occupancy Read Trace) to determine the occurrence count of each target sequence and the number of reads contributing to each occurrence (score). Additionally, the script processes information about the number of reads used to generate each occurrence, which appears to be encoded in sequence identifiers (ID) in the SORT_results.tab file. Certain filters are applied to determine which sequences are considered valid for further analysis. For example, you can specify a minimum read count (-C) and a minimum read pattern count (-D). You can also choose to ignore annotations in sequence identifiers (-G). If a list of patterns is provided (-P), the script divides sequences into groups based on these patterns and selects the best sequence from each group, as well as any secondary sequence that meets the filter criteria. Patterns can also be provided to ban sequences (-B), meaning these sequences will be excluded from analysis.

    - **Possible parameters**:
      - `-P, --pattern-list <STRING>`: Comma-separated list of patterns to group genes. Special case __ALL__ to select the top gene.
      - `-G, --ignore-annotations`: Ignore annotations in target identifiers.
      - `-C, --min-read-count <INTEGER>`: Minimum read count threshold for a target to be considered valid (default = 1).
      - `-D, --min-read-patterns <INTEGER>`: Minimum read patterns threshold for a target to be considered valid (default = 1).
      - `-B, --ban-list <STRING>`: Comma-separated list of patterns to ban specific genes`

    - Uses **options** from the configuration in this step such as:
      - `SORT_GROUPS`: Determines sorting groups for primary and secondary data.
      - `BAN_GROUPS`: Patterns not allowed.
      - `MIN_RC`: Minimum read count to continue attempting gene segment assembly.
      - `MIN_RP`: Minimum read pattern count to continue attempting gene segment assembly.
      - `REF_SET`: Reference set used for the initial round of read collection. Contained in the module's reference folder.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/parseSORTresults.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/SORT/R1/SORT_result.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/MATCH/R1.match flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/SORT/R1 -C 15 -D 15 -G -P PB2,PB1,PA/P3,HA/HE,NP,NA/HE,MP,NS -B UNRECOGNIZABLE
      ```

    - **Output**:
      - `R1.txt`

        ```bash
        A_MP	17523	98438
        A_NS	16521	80518
        A_HA_H1	8136	31887
        A_PB2	7677	25275
        A_NA_N1	6329	24145
        A_NP	5391	18242
        A_PA	3592	8496
        A_PB1	2542	5009
        ```

      - `R1-A_HA_H1.fa`

        ```bash
        >C6%1%|INTER
        cagatacaccagtccacgattgcaatgcaacttgacagacacccgagggtgctataaacaccagcctcccatttcaaaatgtacatccgatcacgattgggaaatgtccaaagtatgtaagaagcacaaaattgagactggccacaggatt
        >C60%2%|INTER
        accgaggtatgcattcacaatggaaaaagaagctggatctggtattatcacttcagatacaccagtccacgattgcaatgcaacttgtcagacacccgagggtgctataaacaccagcctcccatttcaaaatgtacatccgatcacgatt
        >C64%14%|INTER{c}
        ctgaccaagaaagtctctatcagaatgcagatgcatatgtttttgtggggacatcaagatacagcaagaagttcaagccggaaatagcagcaagacccaaagtgagggatcaagcagggagaatgaactattactggacactagtagaacc
        ```

7. _fastaExtractor_: Perl script. For each flu fragment found in the sample, extracts its sequences from the reference file used in the BLAT step.

    - **Command line**:

      ```bash
      flu-amd/LABEL_RES/scripts/fastaExtractor.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/GENES/R0.refs <(echo A_HA_H1) > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/GENES/R0-A_HA_H1.ref
      ```

    - **Output**:
      - `GENES/R0-A_HA_H1.ref`

        ```bash
        >A_HA_H1
        ATGAAGGCAATAATACTAGTAGTTCTGCTATATACATTTGCAACCGCAAATGCAGACACATTATGTATAGGTTATCATGCGAACAATTCAACAGACACTGTAGACACAGTACTAGAAAAGAATGTAACAGTAACACACTCTGTTAACCTTCTAGAAGACAAGCATAACGGGAAACTATGCAAACTAAGAGGGGTAGCCCCATTGCATTTGGGTAAATGTAACATTGCTGGCTGGATCCTGGGAAATCCAGAGTGTGAATCACTCTCCACAGCAAGCTCATGG[...]
        ```

#### 1.4. Do ALIGN

##### 1.4.1. Do SAM

8. _interleavedSamples_: Custom Perl script. Distributes the fasta output from _parseSORTresults_(6) into groups based on the number of processes (cpus) specified.

    - Uses **options** from the configuration in this step such as:
      - `SINGLE_LOCAL_PROC`: For parallelization of a task on the computer, IRMA typically runs using the number of logical CPU cores.
      - `ALIGN_PROC`: If grid execution is active and the limit is exceeded, the number of matrix tasks for gene segment alignment step.

    - **Parameters used**:
      - `-G, --groups`: Number of groups to divide sequences into.
      - `-S, --silence`: Silences summary output.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/interleavedSamples.pl --silence flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/SORT/R1-A_NP.fa flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ALIGN/R1-A_NP -G 2
      ```

    - **Output**: Separate fasta files from the original file.

9.  _align2model_: SAM: Sequence Alignment and Modeling Software System Binary Distribution align2model v3.5 (July 15, 2005) compiled 01/13/21_14:19:37. Usage: 

      ```bash
       flu-amd/LABEL_RES/scripts/align2model run_name [-option value]*`

       -modelfile model_file: required
       -db seq_file [-db seq_filen]* one or more sequence files (in this case interleaved reads)
       [-id seqid]*: one or more sequence IDs
       [-pptrim <value>]*: create .ta2m trimmed alignment
      ```

    - **Command line**:

      ```bash
      flu-amd/LABEL_RES/scripts/align2model_Darwin flu-amd/IRMA_RES/ppath/A-Murcia-#hash/ALIGN/R1-A_HA_H1_001 -modelfile flu-amd/IRMA_RES/modules/FLU_AD/profiles/A_HA_H1_hmm.mod -db R1-A_HA_H1_001.fasta
      ```

    - **Output**:
      - `R1-A_HA_H1_0001.a2m`: Alignment files between interleaved reads and the reference using the hmm.

        ```bash
        >C92%1%|INTER{c}
        ..................................................
        .................................-----------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------.-----............
        .........-----------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------..----------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        -----------------------------------------.-----.--
        -----------------------------------------CTTTGGAGT
        GTCTGGAGTAAATGAATCAGCTGACATGAGTATTGGAGTAACAGTGATAA
        AGAACAACATGATAAACAATGACCTTGGACCTGCAACGGCTCAGATGGCT
        CATCAATTGTTCATAAAAGACTACAGATACACATATAGG-----------
        --------------------------------------------------
        --------------------------------------------------
        --------------------------------------------------
        ```

10. _a2mToMatchStats_: Custom Perl script. For each sequence in the alignment, the script performs the following actions:
    - Removes dots (.) from the sequence.
    - Identifies leader and trailer residues in the sequence.
    - Repairs the sequence if necessary, adding leader or trailer residues as specified.
    - Records character frequencies in each alignment column.
    - Generates statistics: The script stores character frequencies in a specific data structure format and saves them in an output file specified as the second argument (R1-A_HA_H1_0001.sto).
    - Option for reference elongation: If the -S or --skip-elongation option is not activated, the script also analyzes leader and trailer residues of each sequence to obtain statistics on them.


    - Uses **options** from the configuration in this step such as:
      - `SKIP_E`: Skip elongation/extension of reference at 5′ and 3′ ends. Used during read collection phase.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/a2mToMatchStats.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ALIGN/R1-A_HA_H1_001 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ALIGN/R1-A_HA_H1_001.sto -S
      ```

    - **Output**:
      - `R1-A_HA_H1_001.sto`: .sto: Stockholm format is a multiple sequence alignment format used by Pfam, Rfam and Dfam, to disseminate protein, RNA and DNA sequence alignments. The alignment editors Ralee, Belvu and Jalview support Stockholm format as do the probabilistic database search tools, Infernal and HMMER, and the phylogenetic analysis tool Xrate. Como está binario no lo puedo ver. Jalview me dice que el formato es antiguo y tampoco me deja verlo.


##### 1.4.2. Create new reference

11. _combineALIGNstats_: Uses base frequency statistics for each alignment position (.sto) and selects the most common base as the consensus base at that position.

    - Uses **options** from the configuration in this step such as:
      - `SKIP_E`: Skip elongation/extension at 5′ and 3′ ends. Used during read collection phase.
      - `MIN_CA`: Minimum count of alleles for alternative reference generation.
      - `MIN_FA`: Minimum frequency for alternative reference generation based on alternative allele or most frequent minority allele. Set to 1 to disable. Used during read collection and final assembly.
      - `DEL_TYPE`: Advanced option. If sites are completely missing during read collection, use reference seed (REF), delete by ambiguity (NNN), or simply delete (DEL). Default behavior is former: uses "NNN" with BLAT and "DEL" in other cases. Can be specified per round with space delimiter.
      - `DENOM`: Specifies the denominator for `-O` option in the script.

    - **Possible parameters**:
      - `--name` (`-N`): Specifies a name for the output file header.
      - `--min-pad-count` (`-M`): Specifies minimum pad count for counting.
      - `--delete-by-ambiguity` (`-A`): Activates deletion by ambiguity.
      - `--skip-elongation` (`-S`): Skips elongation.
      - `--keep-deleted` (`-K`): Keeps deleted using specific reference sequence.
      - `--debug-mode` (`-D`): Activates debug mode.
      - `--count-alt` (`-C`): Specifies alternative count.
      - `--count-freq` (`-F`): Specifies count frequency.
      - `--denominator` (`-O`): Specifies denominator. Defaults to 2. It's a value used to numerically adjust minimum thresholds and other calculations within data processing, ensuring resulting values are suitable for script's specific purposes.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/combineALIGNstats.pl -N A_HA_H1 -C 20 -F 1 -S  flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#has/ALIGN/R*-A_HA_H1_????.sto > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#has/GENES/R1-A_HA_H1.ref
      ```

    - **Output**:
      - `R1-A_HA_H1.ref`: Generates a new fasta reference for each fragment.

        ```bash
        >A_HA_H1
        ATGAAGGCAATACTAGTAGTTATGCTGTATACATTTACAACCGCAAATGCAGACACATTATGTATAGGTTATCATGCGAACAATTCAACAGACACTGTGGACACAGTACTAGAAAAGAATGTAACAGTAACACACTCTGTCAATCTTCTAGAAGACAAGCATAACGGAAAACTATGCAAACTAAGAGGGGTAGCCCCATTGCATTTGGGTCAATGTAACATTGCTGGCTGGATCTTGGGAAATCCAGAGTGTGAATCACTCTCCACAGCAAGATCATGGTCCTACATTGTGGAAACATCTAATCCAGACAATGGAACGTGTTACCCAGGAGATTTCATCAATTATGAGGAGCTAAGAGAGCAATTGAGCTCAGTGTCATCATTTGAAAAGTTTGAAATATTCCCCAAGACAAGTTCATGGCCTAATCATGACTCGGACAATGGTGTAACTGCAGCATGTTCTCACGCTGGAGCAAGGAGCTTCTACAAAAACTTGATATGGCTGGTTAAAAAAGGAAAAACGTACCCAAAGATCAACCAAACCTACATTAATGATAAAGGGAAAGAAATCCTCGTGCTGTGGGGCATTCACCATCCACCCACTATTACTGACCAAGAAAGTCTCTATCAGAATGCAGATGCATATGTTTTTGTGGGGACATCAAGATACAGCAAGAAGTTCAAGCCGGAAATAGCAGCAAGACCCAAAGTGAGGGATCAAGCAGGGAGAATGAACTATTACTGGACACTAGTAGAACCGGGAGACAAAATAACATTCGAAGCAACTGGTAATCTAGTGGCACCGAGGTATGCATTCACAATGGAAAAAGAAGCTGGATCTGGTATTATCACTTCAGATACACCAGTCCACGATTGCAATGCAACTTGTCAGACACCCGAGGGTGCTATAAACACCAGCCTCCCATTTCAAAATGTACATCCGATCACGATTGGGAAATGTCCAAAGTATGTAAGAAGCACAAAATTGAGACTGGCCACAGGATTGAGGAATGTCCCGTCTATTCAATCTAGGGGCCTATTCGGGGCCATTGCTGGCTTCATCGAAGGGGGGTGGACAGGGATGGTAGATGGATGGTACGGTTATCACCATCAAAATGATCAGGGATCAGGATATGCAGCCGATCTGAAGAGCACACAAAATGCCATTGATAAGATTACCAACAAAGTAAATTCTGTTATTGAAAAGATGAATACACAGTTCACAGCAGTTGGTAAAGAGTTCAACCACCTTGAAAAAAGAATAGAGAATCTAAATAAAAAGGTTGATGATGGTTTCCTGGACGTTTGGACTTACAATGCCGAACTGCTGATTCTACTGGAAAATGAAAGAACTTTGGACTATCACGATTCAAATGTGAAGAACTTGTATGAAAAAGTAAGACACCAGTTAAAAAACAATGCCAAGGAAATTGGAAACGGCTGCTTTGAATTTTACCACAAATGCGACAACACATGCATGGAAAGTGTCAAGAATGGAACTTATGACTACCCAAAATACTCAGAGGAAGCAAAATTAAACAGAGAAAAAATAGATGGAGTAAAGCTGGACTCAACAAGGATTTACCAGATTTTGGCAATCTATTCAACTGTTGCCAGTTCATTGGTACTGGTAGTCTCCCTGGGGGCAATCAGCTTCTGGATGTGCTCTAATGGGTCTCTACAGTGTAGAATATGTATTTAA
        ```

      - `R1.refs`: Generates a single .refs file for all `R1-*.ref` files for each round.

> [!NOTE]
> **Points 7 to 11 are performed for each flu fragment.**
>
> **Points 4 to 11 of doRound are repeated until reaching $MAX_ROUNDS**

### 2. Do post-processing

#### 2.1. Inflate to fastq files:

12. _finalRefs_: This script processes .ref files from step 11 and selects the most recent sequence for each gene, ignoring certain annotations or alternative sequences if corresponding options are specified:
    - **Operation**: For each .refs file in the GENES folder:
        1. Opens the file for reading.
        2. Determines the round number (X) based on the file name (RX).
        3. Processes the content of the file fragment by fragment. For each fragment:
            1. Splits the sequence into lines and separates the fragment name from the sequence content.
            2. Converts the sequence to lowercase. Ignores empty sequences.
            3. If exclude-alternative (-X) is specified, ignores sequences with `alt` in the fragment name.
            4. If ignore-annotation (-G) is specified, removes annotations in the fragment name.
            5. Saves the sequence if it's the most recent (highest round number) for that fragment.
    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/finalRefs.pl -G flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/GENES/R[1-9].refs
      ```

    - **Output**: The output is not a file but stdout used for iteration in subsequent steps. It sorts fragment names alphabetically and prints them along with the highest round number found for each fragment.

        ```bash
        R2-A_HA_H1 R2-A_MP R2-A_NA_N1 R2-A_NP R2-A_NS R2-A_PA R2-A_PB1 R2-A_PB2%
        ```

13. For each of these fragments, concatenate the fasta files from _parseSORTresults_ (6).

    - **Command line**:

      ```bash
      cat flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/SORT/R*-A_HA_H1.fa > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/A_HA_H1.fa
      ```

    - **Output**:
      - `A_HA_H1.fa`

        ```bash
        >C2%1%|INTER{c}
        taaagctggactcaacaaggatttaccagattttggcaatctattcaactgttgccagttcattggtactggtagtctccctgggggcaatcagcttctggatgtgctctaatgggtctctacagtgt...
        >C10%1%|INTER{c}
        gtcagacacccgagggtgctataaacaccagcctcccatttcaaaatgtacatccgatcacgattgggaaatgtccaaagtatgtaagaagcacaaaattgagactggccacaggattgaggaat...
        >C32%1%|INTER
        gagtgtggatcactctccacagcaagatcatggtcctacattgtggaaacatctaatccagacaatggaacgtgttacccaggagatttcatcaattatgaggagctaagagagcaattgagctc...
        >C50%4%|INTER
        gacaagcataacggaaaactatgcaaactaagaggggtagccccattgcatttgggtcaatgtaacattgctggctggatcttgggaaatccagagtgtgaatcactctccacagcaagatcatg...
        >C152%2%|INTER{c}
        attactggacactagtagaaccgggagacaaaataacattcgaagcaactggtaatctagtggcaccgaggtatgcattcacaatggaaaaagaagctggatctggtattatcacttcagataca...
        >C180%1%|INTER{c}
        ctattcaactgttgccagttcattggtactggtagtctccctgggggcaatcagcttctggatgtgctctaatgggtctctacagtgtagaatatgtatttaacattagaatttcagaatcatg...
        ```

14. _xlfate_: Launches the same script as in step (3), but in reverse, instead of converting fastqs to fastas, it converts fastas to fastqs. Uses the `R0.xfl` file from step 3, the fasta generated by concatenating the fastas from _parseSORTresults_ (6). Takes the `R0.xfl` table file to determine how to reconstruct the sequences. Uses sequences from the A_HA_H1.fa file and processes them to obtain the original sequences in FASTQ format. Applies a reverse complement transformation (-R) to the sequences during inflation. Generates a FASTQ file (A_HA_H1.fastq) with inflated sequences.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/xflate.pl -I -Q -R flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/R0.xfl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/A_HA_H1.fa > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/A_HA_H1.fastq
      ```

    - **Parameters**:
        - `-I or --inflate`: Indicates inflation should be performed.
        - `-R or --reverse-complement-inflate`: Performs inflation with reverse complement sequence.
        - `-Q or --fastQ`: Specifies expecting input in FASTQ format and output in the same format.

    - **Output**:
      - `A_HA_H1.fastq`

        ```bash
        @M03352:453:000000000-LBGBK:1:2102:11056:2651 2:N:0:GACGAGATTA+ACATTATCCT
        ctaatgttaaatacatattctacactgtagagacccattagagcacatccagaagctgattgcccccagggagactaccagtaccaatgaactggcaacagttgaatagattgccaaaatctggtaaatccttgttgagtccagcttta
        +
        CCCCCFFFFFFFGGGGGGGGGGHHHHHHHHHHHHGHHHHHHHHHHHHHHHHHHHHHHHHHHGIHHGGGGGGGGGGHHHHHHHHHHHHHHGHHHHHHHHHHHHHHHHHGHHHHHHHHHHHHHHGGGHHHHHHHHHHHHHHHGHHHHHHHH
        @M03352:453:000000000-LBGBK:1:1109:13088:14839 1:N:0:GACGAGATTA+ACATTATCCT
        ggcccctagattgaatagacgggacattcctcaatcctgtggccagtctcaattttgtgcttcttacatactttggacatttcccaatcgtgatcggatgtacattttgaaatgggaggctggtgtttatagcaccctcgggtgtctgac
        +
        >AABCCCCFFFFGGGGGGGGGGGGGGHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHGHHHHHGGGGGHHHHHHHHHGHHHHHHHGGGHHGHHHGHHHHHHHHHGHHGGGHFGHHHHH
        @M03352:453:000000000-LBGBK:1:2102:15694:7060 1:N:0:GACGAGATTA+ACATTATCCT
        gagtgtggatcactctccacagcaagatcatggtcctacattgtggaaacatctaatccagacaatggaacgtgttacccaggagatttcatcaattatgaggagctaagagagcaattgagctcagtgtcatcatttgaaaagtttgaaa
        +
        CCCCBFFCFFFFGGGGGGGGGGHHHHHHHHHHHHHHHHHHHHHHHGHHHHHHHHHHHHIHHGFGFFGHHHHGGHHHHHHHHHGHGHHHHHHHHHGHHHHHHHHHGHHHHHHHHGHHHHHGHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH
        @M03352:453:000000000-LBGBK:1:2112:20116:13489 2:N:0:GACGAGATTA+ACATTATCCT
        gagcgaaagcaggggaaaacaaaagcaacaaaaatgaaggcaatactagtagttatgctgtatacatttacaaccgcaaatgcagacacattatgtataggttatcatgcgaacaattcaacagacactgtgg
        +
        ABBBBBBBBFFFGGGGGGGGGGHHHHHHHHHHHHHHHHHHHHHHGHHGHHHHHHHHHHHHHHHHHHGHGHHHFCEGFGGGHFGGHHHHHHHHHHGGHHHHHHHHGHHGHHGGGGGGHHHHHHHHHHHGHHHHH
        ```

15. Write the first two lines of the file output from (11) _combineALIGNstats_ into another file:

```bash
head -n2 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/GENES/R2-A_HA_H1.ref > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_HA_H1.ref
```

This way, the ASSEMBLY folder contains the .fasta file from step (13), the .fastq file with reads from xflate (14), and the .ref file from this step.

    ```bash
    $ ls flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/:  
    A_HA_H1.fa  A_HA_H1.fastq F1-A_HA_H1.ref
    A_MP.fa     A_MP.fastq    F1-A_MP.ref
    A_NP.fa     A_NP.fastq    F1-A_NP.ref
    A_NS.fa     A_NS.fastq    F1-A_NS.ref
    A_PB1.fa    A_PB1.fastq   F1-A_PB1.ref
    A_PB2.fa    A_PB2.fastq   F1-A_PB2.ref
    A_PA.fa     A_PA.fastq    F1-A_PA.ref
    A_NA_N1.fa  A_NA_N1.fastq F1-A_NA_N1.ref
    ```

#### 2.2. Assemble report:

16. _interleavedSamples_: Same process as (4) and (8). Distributes the fastq output from _xflate_ (14) into groups based on the number of processes (cpus) specified.

    - **Parameters used**:
      - `-G, --groups`: Number of groups to divide the sequences into.
      - `-P, --by-read-pairs`: Use FASTQ format and interleave by molecular ID read pairs.
      - `-U, --underscore-header`: Replace spaces in headers with underscores.
      - `-S, --silence`: Silence summary output.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/interleavedSamples.pl --silence --underscore-header -G 2 -P flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/A_HA_H1.fastq flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/A_HA_H1
      ```

    - **Output**:
      - `A_HA_H1_0001.fastq`, `A_HA_H1_0002.fastq`...

##### 2.2.1. Refine assembly:

  - Uses **options** from configuration in this step such as:
    - `SILENCE_COMPLEX_INDELS`: Optionally silence reads with 4 or more indels in any final assembly round where detected. Complex indels may be spurious and affect consensus. Not recommended for long-read technology.
    - `ASSEM_PROG`: Program for final assembly stage, optimized based on assembly score. Uses SSW or MINIMAP2, where the latter integrates all features except multiple chains. SSW is used in this example.
    - `SSW_{options}/MM2_{options}`: Final assembly program options:
      - `SSW_M`: Smith-Waterman local alignment match score (or MM2).
      - `SSW_X`: Smith-Waterman local alignment mismatch penalty (or MM2).
      - `SSW_O`: Smith-Waterman local alignment gap open penalty (or MM2).
      - `SSW_E`: Smith-Waterman local alignment gap extension penalty (or MM2).
      - `MM2_A`: Minimap2 local alignment match score (default SSW_M).
      - `MM2_B`: Minimap2 local alignment mismatch penalty (default SSW_X).
      - `MM2_O`: Minimap2 local alignment gap open penalty (default SSW_O).
      - `MM2_E`: Minimap2 local alignment gap extension penalty (default SSW_E).
    - `MAX_ITER_ASSEMBLY`: Maximum number of iterations for final assembly phase.

17. _ssw_Darwin/Linux_: The manual is available [here](https://github.com/mengyao/Complete-Striped-Smith-Waterman-Library). Striped Smith-Waterman is an optimized implementation of the Smith-Waterman algorithm for local sequence alignment. This implementation uses a technique called "striping" to enhance computational efficiency and can leverage SIMD (Single Instruction, Multiple Data) instructions for parallelizing operations, thus speeding up the alignment process.
    - Uses **options** from configuration in this step such as:
      - `SILENCE_COMPLEX_INDELS`: Optionally silence reads with 4 or more indels in any final assembly round where detected. Complex indels may be spurious and affect consensus. Not recommended for long-read technology.

    - **Parameters used**:
      - `-s`: Output in SAM format.
      - `-c`: Return alignment path. This means the output will include the path of how sequences align.
      - `-h`: Include header in SAM output.
      - `-r`: Select best alignment between the original read alignment and reverse complement read alignment. Useful for sequence reads that may align better in the reverse direction.
      - `-x 5`: Set weight for mismatches.
      - `-m 2`: Set weight for matches.
      - `-o 10`: Set weight for gap opening.
      - `-e 1`: Set weight for gap extension.

    - **Command line**:

      ```bash
      ssw_Darwin -s -c -h -r -x 5 -m 2 -o 10 -e 1 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_MP.ref flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/A_MP_001.fastq > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_MP-002.sam
      ```

    - **Output**:
      - `F1-A_MP-001.sam`

        ```bash
        @HD     VN:1.4  SO:queryname
        @SQ     SN:A_MP LN:982
        M03352:453:000000000-LBGBK:1:2109:10171:6144_1:N:0:GACGAGATTA+ACATTATCCT        0       A_MP    402     5       149M    *       0       0 gatgggaacagtgaccacagaagctgctttcggtctagtttgtgccacttgtgaacagattgctgattcacagcatcggtctcacagacaaatgcctactaccacaaatccattaatcaggcatgaaaacagaatggtgctggctagca   BBBBBBFFBFF@EFGGGGGGGGHHHHHHHHHGGGGGGFHHHHHHHHHHHHHHHHFHHHHHHGHHHGHHHHHHHHHHHGGGGGHHHHHHHHHHHHHHHHHHHHHHHHEGGHHHHHHHHHHGHHHHHHHGHHHFFHHDGHHHHGHGHHFHH   AS:i:291        NM:i:1  ZS:i:211
        M03352:453:000000000-LBGBK:1:2114:13087:12821_1:N:0:GACGAGATTA+ACATTATCCT       0       A_MP    257     5       151M    *       0       0 gaaatggggacccgaacaacatggatagagcagttagactatacaagaaactcaaaagagaaataacgttccatggggccaaagaagtgtcactaagctattcaactggtgcacttgcaagttgcatgggcctcatatacaacaggatggg ABBBBFF@ABBBGGEEGCGGFGGHHHHHHHHHHHHGGHHHHHHHHHHHHHHHHHHHHHGHHHGHHHHGHHGHHHHHHGGGGHHHGHHHHHHHHHHHFHHHHHGHHHHHHHHGHHHHHHHHHHHHHHHHHHHHGHHHHHHHHHHHGHHHHGH AS:i:302        NM:i:0  ZS:i:219
        ```

18. _winnowSAM_: The script is designed to filter and select the best alignments from a SAM file based on alignment scores. It opens the sam file and reads it line by line. It saves the header lines starting with @. It splits each alignment line into its components and analyzes the CIGAR string to count matches or extracts the alignment score from the AS field. It compares scores and saves the best alignment for each read. It overwrites F1-A_MP-002.sam with filtered alignments, including original headers.

    - **Possible parameters**:
      - `-M or --use-matches`: Use matches count in CIGAR for scoring.
      - `-I or --in-place`: Replace original SAM file with filtered SAM file.
      - `-P or --interleaved-pairs`: Append suffix to read names to indicate interleaved pairs.
      - `-F <number> or --score-field <number>`: Specify which field to use for alignment score. Additional fields in SAM start at index 12.

    - **Command line**:

      ```bash
      flu-amd/IRMA_RES/scripts/winnowSAM.pl -I flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_MP-002.sam
      ```

    - **Output**:
      - The same file `F1-A_MP-002.sam` but with filters applied.

19. _samStats_: This script analyzes a SAM file aligned against a reference sequence and generates alignment statistics. It reads the reference sequence from the .ref file and reads through the resulting .sam file from winnowSAM.pl (18). It skips header lines, splits each line into components, analyzes the CIGAR string to determine matches, insertions, and deletions, adjusts complex indels if `-S` option is specified (not specified in IRMA by default), creates a table to store statistics on aligned bases and insertions, counts aligned bases, deleted positions, and inserted bases at each position of the reference, and stores generated statistics in the .sto file.

     - Uses **options** from configuration in this step such as:
       - `ASSEM_REF`: Sorts the reference set (`REF_SET`) if necessary and uses it to start the final assembly instead of starting with read collection results.

     - **Possible parameters**:
       - `--ignore-annotation | -G`: Ignores annotation in reference names.
       - `--silence-complex-indel | -S`: Silences or adjusts complex indels (insertions and deletions).

     - **Command line**:

       ```bash
       flu-amd/IRMA_RES/scripts/samStats.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_MP.ref flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_MP-002.sam flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_MP-002.sto
       ```

     - **Output**:
       - `F1-A_MP-002.sto`: The .sto format is explained in section (10).

20. _catSAMfiles_: This script is designed to concatenate multiple SAM files into one, ensuring that header lines (starting with @) are included only once in the resulting file.

     - **Command line**:

       ```bash
       flu-amd/IRMA_RES/scripts/catSAMfiles.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_PB1-*.sam > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_PB1.sam
       ```

     - **Output**:
       - `F1-A_PB1.sam`: SAM file containing the concatenation of the previous files.

21. _samtools_: Tools for alignments in the SAM format. Samtools view to convert from sam to bam.

     - **Command line**:

       ```bash
       flu-amd/IRMA_RES/scripts/samtools_Darwin view -bS flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_PB1.sam > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_PB1.bam 2> flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_PB1.bam.log
       ```

     - **Output**:
       - `F1-A_PB1.bam`
       - `F1-A_PB1.bam.log`: This file is deleted.

22. _scoreSAM.pl_: Calculates the sum of alignment scores from a SAM file. It reads the sam file, skips header lines, and for each line, splits it into tab-separated fields. It extracts the score field (field 12, index 11), further splits this field into subfields, and takes the third subfield as the score. The sum of this field for all lines in the sam file is the final score.

     - **Command line**:

       ```bash
       flu-amd/IRMA_RES/scripts/scoreSAM.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_PB1.sam
       ```

     - **Output**: It does not generate a file but uses the score within the pipeline. Using the example .sam output from section (17):
       - Initial fields:
         - AS:i:291
         - AS:i:302
       - We take 291 and 302
       - We sum them, so the score is 593

This is done because it will store the score from the .sam file into a variable, and as long as the score from the new .sam file is worse (smaller) than the previous one, it will proceed with steps (23) to (XX).

23. _combineSAMstats_: This script combines multiple .sto format statistics files generated from SAM files and builds a consensus sequence and an alternate sequence based on provided thresholds and parameters. The script loads the reference (e.g., F1-A_NS.ref) to obtain the reference sequence (e.g., A_NS), combines statistics from all provided .sto files (F1-A_NS-0001.sto, F1-A_NS-0002.sto) into an aggregated data structure, calculates the consensus sequence based on base frequencies at each position of the reference sequence, and constructs an alternate sequence if the frequency of an alternate allele exceeds specified thresholds. It alters insertion if the frequency is ≥ 0.25 and coverage is ≥ 1, deletion if the frequency is ≥ 0.60 and coverage is ≥ 1, and changes alternate allele if the frequency is ≥ 1 and variant count is ≥ 20. The script prints the consensus sequence into .ref and if there's a significant alternate sequence, it also prints it in the same file.

     - **Possible parameters**:
       - `-N`: Name of the consensus sequence.
       - `-I`: Insertion frequency threshold to alter consensus (default 0.15 in Perl script but launched as 0.25 by default in IRMA).
       - `-D`: Deletion frequency threshold to alter consensus (default 0.75 in Perl script but launched as 0.60 by default in IRMA).
       - `-i`: Insertion coverage depth threshold (default 1).
       - `-d`: Deletion coverage depth threshold (default 1).
       - `-A`: Frequency threshold to change an alternate allele from reference (default off in script but turned on (1) in IRMA).
       - `-C`: Variant count to change an alternate allele from reference (default off in script but launched with value 20 in IRMA).
       - `-S`: Saves aggregated statistics in a .sto file.
       - `-M`: Marks deletions in consensus sequence with '-'.
       - `-E`: Minimum coverage support to mask dropouts with flanking regions as 'N'.

     - Uses configuration **options** in this step such as:
       - `INS_T`: Minimum insertion frequency to modify reference. Set to 1 to disable. Used during final assembly phase. Frequency is the count of the most frequent insertion after position P divided by the average coverage depth at positions P and P+1.
       - `DEL_T`: Minimum deletion frequency to modify reference. Set to 1 to disable. Used during final assembly. Coverage depth zero is always removed.
       - `MIN_FA`: Minimum frequency for alternate reference generation based on alternate allele or most frequent minor allele. Set to 1 to disable. Used during read collection and final assembly.
       - `MIN_CA`: Minimum allele count for alternate reference generation.
       - `INS_T_DEPTH`: Minimum insertion coverage depth to modify reference. Set to 0 to disable. Used during final assembly phase. Coverage depth is the count of the most frequent insertion after position P divided by the average coverage depth at positions P and P+1.
       - `DEL_T_DEPTH`: Minimum deletion coverage depth to modify reference. Set to 0 to disable. Used during final assembly. Coverage depth zero is always removed.
       - `MIN_DROPOUT_EDGE_DEPTH`: Minimum coverage depth support to mask dropouts with flanking regions as 'N'.

     - **Command line**:

       ```bash
       flu-amd/IRMA_RES/scripts/combineSAMstats.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_NS.ref -N A_NS -I 0.25 -D 0.60 -A 1 -C 20 -i 1 -d 1 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_NS-0001.sto flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F1-A_NS-0002.sto > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F2-A_NS.ref
       ```

     - **Output**:
       - `F2-A_NS.ref`: This file may or may not be the same as `F1-A_NS.ref`

        ```bash
        >A_NS
        ATGGAATCCAACACCATGTCAAGCTTTCAGGTAGACTGTTTTCTTTGGCATATTCGCAAGCGATTTGCAGACAATGGATTGGGTGATGCCCCATTCCTCGATCGGCTACGCCGAGATCAAAAGTCCTTAAAAGGAAGAGGCAACACCCTTGGCCTCGACATCAAAACAGCCACTCTTGTTGGGAAACAAATTGTGGAATGGATTTTGAAAGAGAAATCCAGCGAGACACTTAGAATGGCAATTGCATCTATACCTACTTCGCGTTACATTTCTGACATGACCCTCGAGGAAATGTCACGAGACTGGTTCATGCTTATGCCTAGGCAAAAGATAATAGGCCCTCTTTGCGTGCGATTGGACCAGGCGGTCATGGATAAGAACATAGTACTGGAAGCAAACTTCAGTGTAATCTTCAACCGATTAGAGACCTTGATACTACTAAGGGCTTTCACTGAGGAGGGAACAATAGTTGGAGAAATTTCACCATTACCTTCTCTTCCAGGACATACTTATGAGGATGTCAAAAATGCAATTGGGGTCCTCATCGGAGGACTTGAGTGGAATGGTAACACGGTTCGAGTCTCTGAAAATATACAGAGATTCGCTTGGAGAAGCTGTGATGAGAATGGGAGACCTTCACTACCTCCAGAGCAGAAATGAGAAATGGCGGGAACAATTGGGACAGAAATTTGAGGAAATAAGATGGTTAATTGAAGAAATACGACACAGATTGAAAGCGACAGAAAATAGTTTCGAGCAAATAACATTTATGCAAGCCTTACAACTACTGCTTGAAGTAGAGCAAGAGATAAGAGCTTTCTCGTTTCAGCTTATTTAA
        ```

> [!NOTE]
> Steps 17 to 23 are repeated up to `MAX_ITER_ASSEM`.

##### 2.2.2. Call variants:

24. _mergeSAMpairs_: The script is designed to perform reference-based merging of aligned read pairs, including detecting and correcting errors in read pair overlaps parsimoniously. The script performs the following steps: loads the reference sequence .ref from (23), reads alignments from the .sam from (20), merges aligned read pairs found at corresponding positions in the reference, using the reference sequence and base qualities to resolve discrepancies in read overlaps, generates an output .sam file (M2-A_MP-001.sam) containing the merged reads, and generates a statistics file (M2-A_MP-001.stats) containing statistics about the merging process.

     - **Possible Parameters**:
       - `-S or --store-stats`: If present, stores statistics of the merging process in a .stats file.

     - **Command Line**:

       ```bash
       flu-amd/IRMA_RES/scripts/mergeSAMpairs.pl -S flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F2-A_MP.ref flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F2-A_MP-001.sam flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/M2-A_MP-001
       ```

     - **Output**:
       - `M2-A_MP-001.sam`

       ```bash
        @HD     VN:1.4  SO:queryname
        @SQ     SN:A_MP LN:982
        M03352:453:000000000-LBGBK:1:1104:16499:9493_3:N:0:GACGAGATTA+ACATTATCCT        0       A_MP    488     5       198M    *       0       0 GACAAATGGCTACTACCACAAATCCATTAATCAGGCATGAAAACAGAATGGTGCTGGCTAGCACTACGGCAAAGGCTATGGAACAGGTGGCTGGATCGAGTGAACAGGCAGCGGAGGCCATGGAGGTTGCTAATAAGACTAGGCAGATGGTACATGCAATGAGAACTATTGGAACTCATCCTAGCTCCAGTGCTGGTC  AABBAFFFFFFFGGGGGGGGAGHFCGGFEEBBGHHGHHGHFHHFCHHHFF3DFFHHCFFGFFHHFHFEBEHGHHGHFHHHHGHGGFBB@?33FFGGFHGEFHHHHHHDEG?GHFFG@GFFBEHFHGFBFGFHHGHHHHGHHHHHHGHHGGEGHFFFHHGGDG6HHHHGGFDFFGFHFG4FGFGFGGFCFFFACABBBB
       M03352:453:000000000-LBGBK:1:2106:21470:3660_3:N:0:GACGAGATTA+ACATTATCCT        0       A_MP    689     5       218M    *       0       0 GAGATGACCTTCTTGAAAATTTACAGGCCTACCAGAAGCGAATGGGAGTGCAGATGCAGCGGTTCAAATGATCCTCTCGTCATTGCAGCAAACATCATTGGGATCTTGCACCTGATATTGTGGATTACTGATCGTCTTTTTTTCAAATGCATTTATCGTCGCTTTAAATACGGTTTGAAAAGAGGGCCTTCTACGGAAGGAGTGCCTGAGTCCATGAG      1AA>AFFFFB1CGGEF11GFG1GHDEC00AB0FBCFCBGE00AG10GFGGFHF111FG11A//ABFBGHFGAGDGCEGGFGGBGHFFHGHHHGHFHHHHFEGHHHGFGHFG?4HGHFCGEHHHGHEGFBHGHGE1GEFHHHGCHFHGHHHHFGGGEE01FEGA5DD3FFAA23G5DDGGF2DGACF4BHFAAEFCHGDGGF4F4CCFFFCCF5AAAA3
       ```

       - `M2-A_MP-001.stats`

       ```bash
        A_MP    obs     1803152
        A_MP    tmv     890
        A_MP    fmv     1348
        A_MP    dmv     7
        A_MP    insObs  15
        A_MP    insErr  2
       ```

25. _varCallStats_: This script processes .sam files to identify genomic variants relative to a reference sequence. It opens the .ref file (23), reads the reference sequence and stores its length, opens the .sam file from (24), for each alignment in the sam file does the following: extracts information such as query name (qname), flag, reference name (rn), position (pos), sequence (seq), and qualities (qual), adjusts start position in the reference (rpos), processes the CIGAR string to identify alignment operations:
      - M: Match/Mismatch - increments nucleotide counts and their qualities
      - I: Insertion - increments insertion counts and their qualities.
      - D: Deletion - increments deletion counts.
      - S: Soft clipping - adjusts position in query sequence without affecting the reference.
      - H: Hard clipping - handled specifically or ignored depending on the case.
      - N: Skipped region - handled specifically or ignored depending on the case.
      - Variant statistics are stored in a file using the provided prefix (V2-A_MP-001.sto).


      - **Command Line**:

       ```bash
        flu-amd/IRMA_RES/scripts/varCallStats.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F2-A_MP.ref flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/M2-A_MP-001.sam flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/V2-A_MP-001
       ```

26. _catSAMfiles_: Explained in step (20). Uses .sam files from step (24).

     - **Command Line**:

       ```bash
       flu-amd/IRMA_RES/scripts/catSAMfiles.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/M2-A_PB1-*.sam > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.sam
       ```

     - **Output**:
       - `A_PB1.sam`

27. _getPairingStats_: The script converts statistics stored in storable format into a tabular result table. Uses .stats files generated in (24) as input. For each line in the .stats files, breaks down the information into rn (first column, fragment name), key (second column, statistic name), and value (third column, statistic value), accumulates statistic values into a hash table organized by fragment name and key. For each fragment name (rn), calculates the total number of observations (obs), minimum expected variations (tmv), minimum expected fragment errors (fmv), deletion errors (dmv), insertion observations (insObs), and insertion errors (insErr), calculates and formats additional metrics such as expected error rate, minimum expected variation, minimum deletion and insertion error rate, and prints the results in tabular format.

     - **Command Line**:

       ```bash
       flu-amd/IRMA_RES/scripts/getPairingStats.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/M2-A_PB1-*.stats > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-pairingStats.txt
       ```

     - **Output**:
       - `A_PB1-pairingStats.txt`: Saved in `results/sample/tables/A_PB1-pairingStats.txt`

       ```bash
       A_PB1   Observations    162513
       A_PB1   ExpectedErrorRate   0.000935309790601367
       A_PB1   MinimumExpectedVariation   0.00183985281177506
       A_PB1   MinimumDeletionErrorRate   3.07667694276766e-05
       A_PB1   MinimumInsertionErrorRate  0.6
       ```

28. _call_: Performs variant calling. Reads the reference sequence that serves as the base for comparison and variant detection. If additional files with pairing statistics, such as expected error rates, are provided, they are also loaded for use in the analysis. The script combines alignment data, calculates the coverage at each position in the reference sequence, and for each position in the reference sequence, determines if there are genetic variants that may include substitutions, insertions, and deletions. It evaluates each detected variant using criteria such as minimum frequency, minimum count, and minimum quality. It generates a variants file and also produces additional files with detailed information about coverage at each position, which is crucial for interpreting the reliability of the detected variants. It provides global statistics on the reference sequence, such as average coverage and the distribution of variants along the sequence.

    - Uses **options** from the configuration at this step, such as:
      - `MIN_C`: Minimum allele count heuristic for variant calling.
      - `MIN_F`: Minimum frequency heuristic for calling single nucleotide variants.
      - `MIN_FI`: Minimum frequency heuristic for calling insertion variants.
      - `MIN_FD`: Minimum frequency heuristic for calling deletion variants.
      - `MIN_AQ`: Minimum average quality score heuristic for allele calls for insertion and single nucleotide variants.
      - `MIN_TCC`: Minimum coverage depth heuristic (total coverage count) for variant calling.
      - `MIN_CONF`: Minimum confidence level that is not a machine error for single nucleotide variants. See the formula.
      - `SIG_LEVEL`: Significance level for statistical tests of variant calling.
      - `AUTO_F`: Automatically adjusts MIN_F based on the maximum zero-confidence minor allele. See the formula.

    - **Possible Parameters**:
      - `no-gap-allele` (`-G`): Does not count gap alleles as variants.
      - `min-freq` (`-F <FLT>`): Minimum frequency for a variant to be processed. Default is 0.01.
      - `min-insertion-freq` (`-I <FLT>`): Minimum frequency for insertions.
      - `min-deletion-freq` (`-D <FLT>`): Minimum frequency for deletions.
      - `min-count` (`-C <INT>`): Minimum count of a variant. Default is 2.
      - `min-quality` (`-Q <INT>`): Minimum average quality of the variant. Default is 20.
      - `min-total-col-coverage` (`-T <INT>`): Minimum non-ambiguous column coverage. Default is 2.
      - `print-all-sites` (`-P`): Prints all variants.
      - `conf-not-mac-err` (`-M <FLT>`): Minimum confidence allowed for machine errors. Default is 0.5.
      - `sig-level` (`-S <FLT>`): Significance level to determine if a variant is not a machine error.
      - `paired-error` (`-E <FILE>`): File with paired error estimates.
      - `auto-min-freq` (`-A`): Automatically finds a minimum frequency heuristic.
      - `call-table` (`-B <FILE>`): Call table specified in the file, format: <POS>[TAB]<ALLELE>.

    - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/call.pl -P -G -C 2 -F 0.008 -I 0.005 -D 0.005 -Q 24 -T 100 -M 0.80 -S 0.999 -A -E flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-pairingStats.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/F2-A_PB1.ref flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/ASSEMBLY/V2-A_PB1-*.sto
      ```

    - **Output**:
      - `A_HA_H1-vars.sto`
      - `A_HA_H1-pats.sto`
      - `A_HA_H1-variants.txt`: Information about the variants found.

        ```bash
        Reference_Name	Position	Total	Consensus_Allele	Minority_Allele	Consensus_Count	Minority_Count	Consensus_Frequency	Minority_Frequency	Consensus_Average_Quality	Minority_Average_Quality	ConfidenceNotMacErr	PairedUB	QualityUB
        A_HA_H1	43	1121	G	A	999	122	0.891168599464764	0.108831400535236	37.2752752752753	37.5491803278689	0.998384422674942	0.00706850138659059	0.00579035169071734
        A_HA_H1	583	1955	G	A	1934	18	0.98925831202046	0.00920716112531969	36.6302998965874	30.5555555555556	0.904430634851244	0.00470881990835601	0.00505203224754183
        A_HA_H1	1383	2193	C	A	2139	54	0.975376196990424	0.0246238030095759	37.7512856474988	37.7037037037037	0.993109122944132	0.00435574846233817	0.00316704053982037
        ```

      - `A_HA_H1-coverage.txt`: Coverage information by position.

        ```bash
        Reference_Name	Position	Coverage Depth	Consensus	Deletions	Ambiguous	Consensus_Count	Consensus_Average_Quality
        A_HA_H1	1	560	A	0	0	560	37.7589285714286
        A_HA_H1	2	561	T	0	0	561	38.1122994652406
        A_HA_H1	3	561	G	0	0	561	37.8823529411765
        A_HA_H1	4	561	A	0	0	561	37.9286987522282
        ```

      - `A_HA_H1.fasta`: Generated consensus sequence.

        ```bash
        >A_HA_H1
        ATGAAGGCAATACTAGTAGTTATGCTGTATACATTTACAACCGCAAATGCAGACACATTATGTATAGGTTATCATGCGAACAATTCAACAGACACTGTGGACACAGTACTAGAAAAGAATGTAACAGTAACACACTCTGTCAATCTTCTAGAAGACAAGCATAACGGAAAACTATGCAAACTAAGAGGGGTAGCCCCATTGCATTTGGGTCAATGTAACATTGCTGGCTGGATCTTGGGAAATCCAGAGTGTGAATCACTCTCCACAGCAAGATCATGGTCCTACATTGTGGAAACATCTAATCCAGACAATGGAACGTGTTACCCAGGAGATTTCATCAATTATGAGGAGCTAAGAGAGCAATTGAGCTCAGTGTCATCATTTGAAAAGTTTGAAATATTCCCCAAGACAAGTTCATGGCCTAATCATGACTCGGACAATGGTGTAACTGCAGCATGTTCTCACGCTGGAGCAAGGAGCTTCTACAAAAACTTGATATGGCTGGTTAAAAAAGGAAAAACGTACCCAAAGATCAACCAAACCTACATTAATGATAAAGGGAAAGAAATCCTCGTGCTGTGGGGCATTCACCATCCACCCACTATTACTGACCAAGAAAGTCTCTATCAGAATGCAGATGCATATGTTTTTGTGGGGACATCAAGATACAGCAAGAAGTTCAAGCCGGAAATAGCAGCAAGACCCAAAGTGAGGGATCAAGCAGGGAGAATGAACTATTACTGGACACTAGTAGAACCGGGAGACAAAATAACATTCGAAGCAACTGGTAATCTAGTGGCACCGAGGTATGCATTCACAATGGAAAAAGAAGCTGGATCTGGTATTATCACTTCAGATACACCAGTCCACGATTGCAATGCAACTTGTCAGACACCCGAGGGTGCTATAAACACCAGCCTCCCATTTCAAAATGTACATCCGATCACGATTGGGAAATGTCCAAAGTATGTAAGAAGCACAAAATTGAGACTGGCCACAGGATTGAGGAATGTCCCGTCTATTCAATCTAGGGGCCTATTCGGGGCCATTGCTGGCTTCATCGAAGGGGGGTGGACAGGGATGGTAGATGGATGGTACGGTTATCACCATCAAAATGATCAGGGATCAGGATATGCAGCCGATCTGAAGAGCACACAAAATGCCATTGATAAGATTACCAACAAAGTAAATTCTGTTATTGAAAAGATGAATACACAGTTCACAGCAGTTGGTAAAGAGTTCAACCACCTTGAAAAAAGAATAGAGAATCTAAATAAAAAGGTTGATGATGGTTTCCTGGACGTTTGGACTTACAATGCCGAACTGCTGATTCTACTGGAAAATGAAAGAACTTTGGACTATCACGATTCAAATGTGAAGAACTTGTATGAAAAAGTAAGACACCAGTTAAAAAACAATGCCAAGGAAATTGGAAACGGCTGCTTTGAATTTTACCACAAATGCGACAACACATGCATGGAAAGTGTCAAGAATGGAACTTATGACTACCCAAAATACTCAGAGGAAGCAAAATTAAACAGAGAAAAAATAGATGGAGTAAAGCTGGACTCAACAAGGATTTACCAGATTTTGGCAATCTATTCAACTGTTGCCAGTTCATTGGTACTGGTAGTCTCCCTGGGGGCAATCAGCTTCTGGATGTGCTCTAATGGGTCTCTACAGTGTAGAATATGTATTTAA
        ```

      - `A_HA_H1-allAlleles.txt`: Detailed information of all alleles if specified.

        ```bash
        Reference_Name	Position	Allele	Count	Total	Frequency	Average_Quality	ConfidenceNotMacErr	PairedUB	QualityUB	Allele_Type
        A_HA_H1	1	A	560	560	1	37.7589285714286	0.99983246438553	0.0124435770878468	0.0110659777886929	Consensus
        A_HA_H1	2	T	561	561	1	38.1122994652406	0.999845556351199	0.0124246336420352	0.0110121477482314	Consensus
        A_HA_H1	3	G	561	561	1	37.8823529411765	0.999837158645599	0.0124246336420352	0.0110346337279279	Consensus
        A_HA_H1	4	A	561	561	1	37.9286987522282	0.999838887170544	0.0124246336420352	0.0110300087007985	Consensus        
        ```

      - `A_HA_H1-insertions.txt`: Insertion information.

        ```bash
        Reference_Name	Upstream_Position	Insert	Context	Called	Count	Total	Frequency	Average_Quality	ConfidenceNotMacErr	PairedUB	QualityUB
        A_HA_H1	1073	C	gggtgCgacag	FALSE	4	2421	0.00165220983064849	37.75	0.898390386803132	0.131919963401386	0.0029041385652492
        A_HA_H1	1075	AAG	gtggaAAGcaggg	FALSE	4	2415	0.00165631469979296	35.8333333333333	0.842411758103167	0.131947118423443	0.00313203031282448
        A_HA_H1	1409	AA	cagttAAaaaaa	FALSE	2	2201	0.000908677873693776	38.5	0.844550243214266	0.132989542561996	0.00308635119233745
        A_HA_H1	1543	A	cagagAaaaaa	FALSE	11	2898	0.00379572118702553	38.4545454545455	0.962394518779417	0.13004684980104	0.00243292133964378
        ```

##### 2.2.3. doPHASING:

29.  _phase_: This script is designed to perform a parallel phasing algorithm. Phasing is the process of determining which alleles are on the same copy of a chromosome. The script takes three mandatory and two optional arguments to split the phasing work into smaller, manageable tasks, allowing for parallel processing. It uses input files of variants and patterns generated in (28), and generates distance matrices describing the relationship between variants. The script loads variants from `vars.sto` and read patterns from `pats.sto`, organizes variants into a list of "alleles" and "names" ordered by position on the genome, calculates the total number of necessary operations and adjusts array size if greater than O, divides the work into segments based on index and array size, calculates distances between alleles using metrics defined in the script, computes several distance metrics between alleles including MUTUALD (mutual distance), JACCARD (Jaccard coefficient), EXPENRD (expected distance), NJOINTP (normalized joint probability), writes the calculated distance results to corresponding output files.

     - **Possible Parameters**:
       - `-S|--array-size`: Size of the array for parallel jobs (default is 10).
       - `-I|--index`: Specific task index in the array (must be between 1 and array size).

     - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/phase.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-vars.sto flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-pats.sto -S 1 -I 001
      ```

     - **Output**: These files contain distance matrices between alleles, computed with the mentioned metrics.
       - `A_HA_H1-0001-EXPENRD.sqm`
       - `A_HA_H1-0001-JACCARD.sqm`
       - `A_HA_H1-0001-MUTUALD.sqm`
       - `A_HA_H1-0001-NJOINTP.sqm`

        ```bash
        43A	0
        583A	1	0
        ```

       - `A_HA_H1-0002-EXPENRD.sqm`
       - `A_HA_H1-0002-JACCARD.sqm`
       - `A_HA_H1-0002-MUTUALD.sqm`
       - `A_HA_H1-0002-NJOINTP.sqm`

        ```bash
        1383A	1	1	0
        ```

Then concatenates all .sqm from the same gene and distance algorithm into a single file:

  - `A_HA_H1-EXPENRD.sqm`
  - `A_HA_H1-JACCARD.sqm`
  - `A_HA_H1-MUTUALD.sqm`
  - `A_HA_H1-NJOINTP.sqm`

        43A	0
        583A	1	0
        1383A	1	1	0

30. _completeMatrix_: The script takes a lower triangular matrix and completes it to become a full (symmetric) matrix. This script ensures that values in the upper right of the matrix are equal to the corresponding values in the lower left.

    - **Possible Parameters**:
      - `-A, --annot-col`: Indicates that the input matrix has an additional annotation column before the matrix data.

    - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/completeMatrix.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-EXPENRD.sqm
      ```

    - **Output**:
      - `A_PB2-EXPENRD.sqm`

        ```bash
        43A	0	1	1
        583A	1	0	1
        1383A	1	1	0
        ```

31. _phases_: The R script performs clustering using the single linkage algorithm based on a distance matrix. It reads the .sqm file and converts the data into a distance matrix, performs single linkage clustering using the specified cutoff value, assigns data to clusters based on the clustering tree structure, prints the cluster assignments to standard output.

    - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/phases.R flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-EXPENRD.sqm
      ```

    - **Output**: It does not generate a result file; output is redirected to step (32).

32. _addPhaseField_: The script adds a phase field to a variants table based on calculated phase data. It takes two input files: a variants table (28) and a phase file (31), and produces an updated variants table with the added phase field. The script reads -variants.txt to get the list of variants and opens and reads the phase file, assigns a phase to each allele and counts occurrences of each phase, sorts phases by frequency and assigns a rank to each phase, finds indices of position and minor allele columns in variants table, for each line in -variants.txt, concatenates position and minor allele to form allele key ($posAllele), looks up corresponding phase in %phaseByAllele and adds this phase (or NULL if not found) to variants line, writes updated variants table with added phase field to A_PB1-variants.txt.

    - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/addPhaseField.pl flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-variants.txt <(flu-amd/IRMA_RES/scripts/phases.R flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-EXPENRD.sqm )
      ```

    - **Output**:
      - `A_MP-variants.txt`: Same file as in (28) but with Phase column added.

        ```bash
        Reference_Name	Position	Total	Consensus_Allele	Minority_Allele	Consensus_Count	Minority_Count	Consensus_Frequency	Minority_Frequency	Consensus_Average_Quality	Minority_Average_Quality	ConfidenceNotMacErr	PairedUB	QualityUB	Phase
        A_MP	572	12067	A	G	11842	223	0.981354106240159	0.0184801524819756	38.0136801216011	34.9417040358744	0.982657007108101	0.00185059354167928	0.001146349353252	1
        ```

##### 2.2.4. Plots

Depending on the number of variants in the `-variants.txt` file, it performs more or fewer steps:
- If the number of lines including the header is greater than 2 (3 or more), it performs steps 33, 34, and 36.
- If the number of lines including the header is greater than 1 (2), it performs steps 33 and 36.
- If the number of lines including the header is 1 (0 variants), it performs steps 35 and 36.

33. _coverageDiagram_: Generates the coverage diagram as in (35).

    - **Command Line**:

      ```bash
      Rscript --vanilla flu-amd/IRMA_RES/scripts/coverageDiagram.R SAMPLE-NAME A_PB2 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-coverage.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-variants.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-pairingStats.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-coverageDiagram.pdf
      ```

    - **Output**:
      - `A_PB2-coverageDiagram.pdf`

34. _sqmHeatmap_: Generates the EXPENRD.pdf plot.

    - **Command Line**:

      ```bash
      Rscript --vanilla flu-amd/IRMA_RES/scripts/sqmHeatmap.R flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-EXPENRD.sqm flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB2-EXPENRD.pdf
      ```

    - **Output**:
      - `A_PB2-EXPENRD.pdf`

35.  _simpleCoverageDiagram_: Generates the coverage diagram as in (34).

     - **Command Line**:

      ```bash
      Rscript --vanilla flu-amd/IRMA_RES/scripts/simpleCoverageDiagram.R SAMPLE-NAME A_PB1 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-coverage.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-coverageDiagram.pdf
      ```

     - **Output**:
       - `A_PB1-coverageDiagram.pdf`

36. _heuristicDiagram_: Generates the heuristic plot.

    - Uses **options** from the configuration in this step such as:
      - `MIN_AQ`: Minimum average allele quality score heuristic for calling insertion and single nucleotide variants.
      - `MIN_F`: Minimum frequency heuristic for calling single nucleotide variants.
      - `MIN_TCC`: Minimum total coverage count (TCC) heuristic for calling variants.
      - `MIN_CONF`: Minimum confidence that it is not a machine error for single nucleotide variants. See formula.

    - **Command Line**:

      ```bash
      Rscript --vanilla flu-amd/IRMA_RES/scripts/heuristicDiagram.R 24 0.008 100 0.80 flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-allAlleles.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-heuristics.pdf
      ```

    - **Output**:
      - `A_PB1-heuristics.pdf`

##### 2.2.5. Amended consensus

37. _amendConsensus_: The script is designed to modify a reference consensus sequence based on provided variant data. It can be used to adjust the consensus with specific variants, manage frequencies of insertions and deletions, and generate an updated coverage file. The script reads the .fasta reference file generated in (28) to obtain the initial sequence, reads the -variants.txt file from (32) to obtain the list of detected variants and their frequencies, filters the variants according to the specified parameters (-C 2 and -F 0.5), adjusts the reference sequence with the valid variants, replacing the bases at the corresponding positions, applies phase-specific changes if options related to phases are used, creates a modified consensus sequence file with the specified name and prefix (sample name).

    - Uses **options** from the configuration in this step such as:
      - `MIN_C`: Minimum allele count heuristic for calling variants.
      - `MIN_AMBIG`: Minimum SNV call frequency for mixed base calls in the modified consensus folder. Used at the end of the final assembly phase.
      - `SEG_NUMBERS`: Used for renaming the modified consensus, typically module-specific. A comma-delimited list of "key:value" pairs.
      - `SORT_GROUPS`: Determination of sorting groups for primary and secondary data.
      - `NONSEGMENTED`: If SORT_GROUPS is empty and if NONSEGMENTED is enabled, adds `-H` to the options (`--fa-header-suffix`)
      - `MIN_CONS_SUPPORT`: Minimum allele coverage depth to call the plurality consensus; otherwise, "N" is called. Setting this value too high may negatively affect the final corrected consensus. Leaving the value blank turns off the function.
      - `MIN_CONS_QUALITY`: Minimum average allele quality to call the plurality consensus; otherwise, "N" is called. Setting this value too high may negatively affect the final corrected consensus. Leaving the value blank turns off the function.

    - **Possible Parameters**:
      - `-C, --count <INT>`: Minimum total count needed (coverage).
      - `-F, --freq <FLT>`: Minimum needed minor variant frequency.
      - `-d, --deletion-file <STR>`: IRMA generated deletion table.
      - `-D, --min-del-freq <FLT>`: Minimum deletion frequency.
      - `-i, --insertion-file <STR>`: IRMA generated insertion table.
      - `-I, --min-ins-freq <FLT>`: Minimum insertion frequency.
      - `-N, --name <STR>` Name of header and file.
      - `-H, --fa-header-suffix`: Add fasta header as suffix to <name>.
      - `-S, --seg <STR>`: Convert the seg name to number and add to ISA.
      - `-P, --prefix <STR>`: Output prefix for file.
      - `-T, --min-total-depth <INT>`: Minimum non-ambiguous column coverage. Default = 2.
      - `-c, --rewrite-coverage <FILE>`: Re-write the coverage file.
      - `-o, --output-coverage-file <FILE>`: Output coverage file, otherwise replaces file exactly.
      - `--min-consensus-support <INT>`: Minimum plurality count to call consensus, otherwise 'N'.
      - `--min-consensus-quality <FLT>`: Minimum plurality average quality to call consensus, otherwise 'N'.
      - `--a2m-reference`: Interpret reference as an align to module fasta file.
      - `-R, --replace-not-ambiguate`: Replace non-ambiguated bases.
      - `-B, --belong-to-phase <INT>`: Replace bases belonging to specified phase.

    - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/amendConsensus.pl -P flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/amended_consensus flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.fasta flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1-variants.txt -C 2 -N SAMPLE-NAME -F 0.5 -S A_PB2:1,B_PB2:2,C_PB2:1,D_PB2:1,A_PB1:2,B_PB1:1,C_PB1:2,D_PB1:2,P3:3,PA:3,HA:4,HE:4,NP:5,NA:6,A_MP:7,B_MP:7,C_MP:6,D_MP:6,A_NS:8,B_NS:8,C_NS:7,D_NS:7
      ```

    - **Output**:
      - `amended_consensus/SAMPLE-NAME_1.fa`

        ```bash
        >SAMPLE-NAME_1
        ATGGAGAGAATAAAAGAGCTGAGAGATCTAATGTCGCAGTCCCGCACTCGCGAGATACTCACTAAGACCACTGTGGACCATATGGCCATCATCAAGAAGTACACATCGGGAAGGCAAGAGAAGAACCCCGCGCTCAGGATGAAGTGGATGATGGCAATGAAATACCCAATTACGGCAGACAAGAGAATAATGGACATAATCCCAGAGAGGAATGAGCAAGGACAGACCCTCTGGAGCAAAATGAACGATGCTGGATCAGACCGAGTGATGGTATCACCTCTGGCAGTAACGTGGTGGAATAGGAATGGTCCAACAACAAATACAGTTCATTACCCTAAGGTATATAAAACTTATTTCGAAAAGGTCGAAAGGTTAAAACATGGTACCTTCGGCCCTGTCCACTTCAGAAATCAAGTTAAAATAAGGAGGAGAGTTGATATGAACCCTGGCCATGCAGATCTCAGTGCCAAGGAGGCACAGGATGTAATTATGGAAGTTGTTTTCCCAAATGAAGTGGGGGCAAGAATACTGACATCAGAGTCACAGCTGGCAATAACAAAAGAGAAGAAAGAAGAGCTCCAGAATTGTAAAATTGCTCCATTGATGGTGGCGTACATGTTAGAAAGAGAATTGGTCCGAAAAACAAGGTTTCTCCCAGTAGCCGGTGGGACAAGCAGTGTTTATATTGAGGTGTTGCACTTGACCCAAGGGACGTGCTGGGAGCAGATGTACACTCCAGGAGGAGAAGTGAGAAATGATGATGTTGACCAAAGCTTGATTATCGCTGCTAGAAACATAGTAAGAAGAGCAGCAGTATCAGCAGATCCATTAGCATCTCTCTTGGAGATGTGCCACAGCACACAGATTGGAGGTGTGAAGATGGTGGACATCCTTAAGCAGAATCCAACTGAGGAGCAAGCCATAGACATATGCAAGGCAGCAATAGGGTTGAGGATTAGCTCATCTTTCAGTTTTGGTGGGTTCACTTTCAAAAGGACAAGCGGATCATCGGTCAAGAAAGAAGAGGAAATGCTAACGGGCAACCTCCAAACACTGAAATTAAGAGTACATGAAGGGTATGAAGAATTCACAATGGTTGGGAGAAGAGCAACAGCTATTCTCAGAAAGGCAACCAGGAGATTGATCCAATTAATAGTAAGCGGGAGAGACGAGCAATCAATAGCTGAAGCAATAATTGTGGCCATGGTTTTCTCACAAGAGGATTGCATGATCAAAGCAGTTAGGGGCGATCTGAACTTTGTCAATAGGGCAAATCAGAGACTGAATCCCATGCACCAACTCTTGAGGCATTTCCAAAAAGATGCAAAGGTGCTTTTCCAGAACTGGGGAATTGAAACCATCGACAATGTGATGGGAATGATAGGAATACTGCCCGACATGACCCCAAGCACGGAGATGTCAATGAGAGGAATAAGAGTCAGCAAGATGGGAGTAGATGAATACTCCAGCACGGAGAGAGTGGTAGTGAGTATTGACCGATTTTTGAGGGTTAGAGATCAAAGAGGAAACGTACTATTGTCTCCTGAAGAAGTCAGTGAAACGCAAGGAATTGAGAAGTTGACAATAACTTATTCGTCATCAATGATGTGGGAAATCAACGGCCCTGAGTCAGTGCTAGTCAACACTTATCAATGGATAATCAGAAACTGGGAAATTGTGAAAATTCAATGGTCACAAGACCCCACAATGTTATACAACAAAATGGAATTTGAACCATTTCAGTCTCTTGTTCCTAAGGCAACCAGAAGCCGGTACAGTGGATTCGTAAGGACACTGTTCCAGCAAATGAGGGATGTGCTTGGGACATTTGACACTGTCCAAATAATAAAACTTCTCCCCTTTGCCGCTGCTCCACCAGAACAGAGCAGGATGCAATTCTCTTCATTAACTGTGAATGTGAGAGGATCAGGGTTAAGAATACTGGTAAGAGGCAATTCTCCAGTATTCAATTATAACAAGGCAACCAAACGACTTACGATTCTTGGAAAGGATGCAGGTGCATTGACTGAAGATCCAGATGAAGGCACATCTGGGGTGGAGTCTGCTGTCCTGAGAGGATTCCTCATTTTAGGCAAAGAAGACAAGAGGTATGGCCCAGCATTAAGCATCAATGAACTGAGCAATCTTGCTAAAGGAGAGAAAGCTAATGTGCTAATTGGGCAGGGGGACATAGTGTTGGTAATGAAACGAAAACGGGACTCTAGCATACTTACTGACAGCCAGACAGCGACCAAAAGAATTCGAATGGCCATCAATTAG
        ```
##### 2.2.6. Post-processing Completion

38.  _vcfGenerator_: The script is designed to generate VCF files from various input files containing genetic variant information. The script generates the VCF header, reads the reference file and the files with variant and indel information, calculates allele frequencies and variant quality, and applies filters based on the provided options to filter out variants that do not meet the specified criteria.

     - Uses **options** from the configuration in this step similar to call.pl (28):
       - `MIN_C`: Minimum allele count heuristic for calling variants.
       - `MIN_F`: Minimum frequency heuristic for calling single nucleotide variants.
       - `MIN_FI`: Minimum frequency heuristic for calling insertion variants.
       - `MIN_FD`: Minimum frequency heuristic for calling deletion variants.
       - `MIN_AQ`: Minimum average quality score heuristic for calling insertion and single nucleotide variants.
       - `MIN_TCC`: Minimum coverage depth heuristic (total coverage count) for calling variants.
       - `MIN_CONF`: Minimum confidence not being a machine error for single nucleotide variants. See formula.
       - `SIG_LEVEL`: The significance level for statistical variant calling tests.
       - `AUTO_F`: Automatically adjusts MIN_F based on the maximum zero-confidence minor allele. See formula.

     - **Possible Parameters**:
       - `-G`, `--no-gap-allele`: Do not count gap alleles as variants.
       - `-F <FLT>`, `--min-freq <FLT>`: Minimum frequency for a variant to be processed. Default = 0.01.
       - `-I <FLT>`, `--min-insertion-freq <FLT>`: Minimum frequency for insertions. Default = value of `--min-freq`.
       - `-D <FLT>`, `--min-deletion-freq <FLT>`: Minimum frequency for deletions. Default = value of `--min-freq`.
       - `-C <INT>`, `--min-count <INT>`: Minimum count of the variant. Default = 2.
       - `-Q <INT>`, `--min-quality <INT>`: Minimum average quality of the variant. Default = 20.
       - `-T <INT>`, `--min-total-col-coverage <INT>`: Minimum non-ambiguous column coverage. Default = 2.
       - `-N`, `--name`: Provide a name for the variants.
       - `-M <FLT>`, `--conf-not-mac-err <FLT>`: Minimum confidence that it is not a machine error. Default = 0.5.
       - `-S <FLT>`, `--sig-level <FLT>`: Significance level (90, 95, 99, 99.9) that the variant is not a machine error.
       - `-E <FILE>`, `--paired-error <FILE>`: File with paired error estimates.
       - `-A`, `--auto-min-freq`: Automatically find the minimum heuristic frequency.
       - `-V <FILE>`, `--rewrite-var-table <FILE>`: Rewrite the variant table in the specified file.

     - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/vcfGenerator.pl -C 2 -F 0.008 -I 0.005 -D 0.005 -Q 24 -T 100 -M 0.80 -S 0.999 -A flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1.fasta flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1-allAlleles.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1-insertions.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1-deletions.txt > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1.vcf
      ```

     - **Output**:
       - `A_HA_H1.vcf`

        ```bash
        ##fileformat=VCFv4.2
        ##fileDate=20240708
        ##source=flu-amd/IRMA_RES/scripts/vcfGenerator.pl -C 2 -F 0.008 -I 0.005 -D 0.005 -Q 24 -T 100 -M 0.80 -S 0.999 -A flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1.fasta flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1-allAlleles.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1-insertions.txt flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1-deletions.txt
        ##reference=flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_HA_H1.fasta
        ##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
        ##INFO=<ID=DPI,Number=.,Type=Integer,Description="Total Insertion Depth">
        ##INFO=<ID=DPD,Number=.,Type=Integer,Description="Total Deletion Depth">
        ##INFO=<ID=AF,Number=A,Type=Float,Description="Allele Frequency">
        ##INFO=<ID=AQ,Number=A,Type=Float,Description="Average Allele Quality">
        ##INFO=<ID=QUB,Number=A,Type=Float,Description="Second-order corrected 99.9% binomial confidence upper bound on quality error estimate">
        ##INFO=<ID=PUB,Number=.,Type=Float,Description="Second-order corrected 99.9% binomial confidence upper bound on paired error estimate">
        ##FILTER=<ID=dp100,Description="Minimum Total Coverage Depth 100">
        ##FILTER=<ID=cnt2,Description="Minimum Minority Allele Count 2">
        ##FILTER=<ID=q24,Description="Minimum Minority Allele Average Quality 24">
        ##FILTER=<ID=fins0.005000,Description="Minimum Insertion Frequency 0.005000">
        ##FILTER=<ID=fdel0.005000,Description="Minimum Deletion Frequency 0.005000">
        ##FILTER=<ID=conf0.800000,Description="Minimum Confidence (Proportion not Machine Error) 0.800000">
        ##FILTER=<ID=sigP99.9,Description="Second-order corrected 99.9% binomial confidence upper bound on paired error estimate">
        ##FILTER=<ID=sigQ99.9,Description="Second-order corrected 99.9% binomial confidence upper bound on quality error estimate">
        ##FILTER=<ID=fsnv0.008000,Description="Minimum Minority Allele Frequency 0.008000">
        #CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO
        A_HA_H1	43	.	G	A	.	PASS	DP=1121;AF=0.1088314005;AQ=37.55;PUB=0.0070685014;QUB=0.0057903517
        A_HA_H1	583	.	G	A	.	PASS	DP=1955;AF=0.0092071611;AQ=30.56;PUB=0.0047088199;QUB=0.0050520322
        A_HA_H1	1383	.	C	A	.	PASS	DP=2193;AF=0.0246238030;AQ=37.70;PUB=0.0043557485;QUB=0.0031670405
        ```

39. _samtools_: Converts the .sam to .bam, sorts it, and indexes it

    - **Command Line**:

      ```bash
      flu-amd/IRMA_RES/scripts/samtools_Darwin view -bS flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.sam > flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.bam 2> flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.bam.log && flu-amd/IRMA_RES/scripts/samtools_Darwin sort flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.bam flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1 >> flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.bam.log && flu-amd/IRMA_RES/scripts/samtools_Darwin index flu-amd/IRMA_RES/ppath/SAMPLE-NAME-#hash/A_PB1.bam
      ```

Afterwards, it moves the files to folders and reorganizes, cleans up, etc.

### 3. Residual assembly

If the residual assembly .fasta file exists, perform a residual assembly, overwrite some variables, and rerun doRound and Post-processing.

### 4. Secondary assembly
If the DO_ASSEMBLY variable is ON, it performs these steps, which include another doRound and another post-processing.
