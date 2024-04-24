# Viralrecon RSV Service

RSV service is quite similar to SARS-COV-2 as both use `viralrecon` but in different ways. If you find difficulties during the analysis, you may find answers in [SARS CoV 2 service description.](https://github.com/BU-ISCIII/BU-ISCIII/wiki/SARS-CoV-2-service)

1. Create the service using `buiscii-tools` and move it to `scratch_tmp`.
    * **Important:** Make sure that the `primers_scheme.bed` and `primers.fasta` files for each reference are correctly assigned and located in `REFERENCES`, you will need them later on. If you only have the sequence of the primers, you can create your own `scheme.bed` file using blast. Check [How to create scheme.bed](https://github.com/BU-ISCIII/BU-ISCIII/wiki/RSV%E2%80%90Service#how-to-create-scheme.bed) for more information.

2. Go to `ANALYSIS`:
   1. Create `samples_ref.txt`:
      Create samples ref (replace `NC_XXXXXX` and `Human` with the references and host requested):

      ```bash
      1. cp samples_id.txt samples_ref.txt
      2. cp samples_id.txt samples_ref2.txt
      3. sed -i 's/$/\tNC_001803.1\tHuman/' samples_ref.txt
      4. sed -i 's/$/\tNC_038235.1\tHuman/' samples_ref2.txt
      5. cat samples_ref2.txt >> samples_ref.txt
      6. rm samples_ref2.txt
      ```

3. Execute the `lablog bash lablog`. This will create folders `YYYYMMDD_ANALYSIS_0X`, one for each host.
4. Go to `ANALYSIS_0X` and modify the `lablog` to suit the new reference (check if the correct `DOC/*.config` and `DOC/*_params.yml` are being used for RSV).
   * You most likely will have references for RSV-A and RSV-B, you will need to include the following lines in `lablog_viralrecon` under the line containing `nextflow run` refering to the files that correspond in each case:

     ```bash
          --primer_bed ../../REFERENCES/Primers_scheme_rsv.bed \
          --primer_fasta ../../REFERENCES/RSV_primers.fasta \
          --nextclade_dataset_name 'rsv_x' \
          --nextclade_dataset false \
          --nextclade_dataset_tag 'YYYY-MM-DDTXX:00:00Z' \
     ```

   * **Note:** As with any viralrecon analysis, make sure to use the latest [pangolin](https://github.com/cov-lineages/pangolin/releases) and [Nextclade](https://github.com/nextstrain/nextclade_data/blob/release/CHANGELOG.md) database and software versions. You may find the latter in the [Galaxy's singularity depot](https://depot.galaxyproject.org/singularity/)
5. Run `bash lablog_viralrecon`. This should create a `_01_` for each reference.
6. Execute `Module load Nextflow singularity` and then run `bash _01_run_NC_XXXXXX_viralrecon.sh`. To execute the second, wait for the first to finish.
7. While the first process is running, go to `_0X_MAG` and run `bash lablog` to generate the kraken tables.
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

### How to create scheme.bed

`scheme.bed` is a file that contains coordinates of where your primers are located inside a reference in a table format. The columns are:

| Column Index  | Name | Description  
| ------------- | ------------- | ------------- |
| 1  | chrom  | The reference genome.  |
| 2  | chromStart  | Start position of the sequence  |
| 3  | chromEnd  | Ending position of the sequence  |
| 4  | name  | Primer name (make sure its the same as `primers.fasta`)  |
| 5  | primerPool  | Primer pool (irrelevant most of the times)  |
| 6  | strand  | Orientation of the strand |

If the reference selected for the service has changed, it is quite likely that your primers now land on different positions of the genome. Therefore, you will need to create a new bed file with the new coordinates.

To do so, you can use blast in order to align your sequences to your reference:
`blastn -num_threads 10 -evalue 1 -task 'blastn-short' -subject /data/bi/references/virus/RSV/your_reference.fasta -query primers.fasta -out blast.txt -outfmt '6 stitle std slen qlen qcovs' -num_alignments 1`

This will generate a series of alignments with your primers sequences in `blast.txt`. Lets add a header so it's easier to understand:

`echo 'stitle    qaccver saccver pident  length  mismatch        gapopen qstart  qend    sstart  send    evalue  bitscoreslen    qlen    qcovs' | cat - blast.txt > blast_mod.txt`

**Note:** You need only one row per primer in your scheme.bed but you will most likely find some queries that don't match exactly with the reference. In these cases you should try to select the ones with the `length` as close to the `qlen` as possible but keeping the `pident` as high as possible too (try to find a balance between the two metrics)

Once you have selected the corresponding lines, you can execute an auxiliar script called `blast_parser.py` (you may find it in `/data/bi/references/auxiliar_scripts/`) to do the rest of the work:
```python3 blast_parser.py blast_mod.txt scheme.bed```

### Common errors

If the set of adapters is changed, you must provide a new primers_scheme.bed and primers.fasta for the service. Keep in mind that if the orientation of the adapters is 5'-3', then you will need to change the sequence of those primers in Reverse (-) to its reverse complementary in order to make sure that they are correctly trimmed by cutadapt. The reason can be found in [Cutadapt documentation](https://cutadapt.readthedocs.io/en/stable/guide.html#paired-end) as each primer has to be associated with each sequencing orientation (R1 or R2) and intrinsecly to its own orientation (the primers sequence can be expressed either 3'-5' or 5'-3') if you pass all the primers in a single multifasta regardless of their orientation inside the genome, you will need to change all the sequences of the primers in one of the orientations (in our case, the Reverse ones) to their reverse complementary sequence.

Viralrecon cannot handle any designations for primer names rather than `_RIGHT` (for + or Forward) and `_LEFT` (for - or Reverse) so make sure that your primers are named accordingly. Example:

```
Primer_scheme.bed
NC_038235.1     1       XX      RSVCombinitial_LEFT     1       +
NC_038235.1     XXXX    XXXX    RSVWGS_4_RIGHT  2       -
NC_038235.1     XXXX    XXXX    RSVWGS_2_LEFT   1       +
NC_038235.1     XXXX    XXXX    RSVWGS_1_RIGHT  2       -

Primers_WGS.fasta
>RSVCombinitial_LEFT
ACGCCCCGGGAAAAAAAAAA
>RSVWGS_4_RIGHT
CATGWTGWYTGGGGGTGAAA
>RSVWGS_2_LEFT
CAAAAWWWWTTTAAAAGGG
>RSVWGS_1_RIGHT
KKKCCAAATTTKKACAGG
```
