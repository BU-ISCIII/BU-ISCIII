# How to perform the Diploid Outbreak service

This guide describes how to perform the **diploid outbreak** service as a member of the ISCIII Bioinformatics Unit, from service creation using the `buisciii-tools` to analysis, result review and delivery to the researcher.

## Index

- [Introduction](#introduction)
  - [Service request and scope](#service-request-and-scope)
  - [Software used](#software-used)
- [Bioinformatics procedure](#bioinformatics-procedure)
  - [1. Create the service](#1-create-the-service)
  - [2. Prepare the reads and copy the service to scratch](#2-prepare-the-reads-and-copy-the-service-to-scratch)
  - [3. Run Sarek](#3-run-sarek)
  - [4. Postprocess the joint VCF](#4-postprocess-the-joint-vcf)
  - [5. Generate the SNP alignment with SNPhylo](#5-generate-the-snp-alignment-with-snphylo)
  - [6. Infer the phylogenetic tree with IQ-TREE](#6-infer-the-phylogenetic-tree-with-iq-tree)
  - [7. Collect whole-genome coverage metrics](#7-collect-whole-genome-coverage-metrics)
- [Review the results](#review-the-results)
- [Prepare the delivery](#prepare-the-delivery)
- [Complete the service with ``buisciii-tools``](#complete-the-service-with-buisciii-tools)
- [Diploid Outbreak report template](#diploid-outbreak-report-template)

## Introduction

### Service request and scope

The service compares whole-genome sequencing data from diploid, non-human isolates, such as *Candida parapsilosis*, to investigate their genomic relationships in an outbreak study. Reads are mapped to a common reference, variants are called jointly and a SNP alignment is used to infer a phylogenetic tree.

Before starting, please check the sample list, organism, reference assembly and the researcher's service message. Establish which isolates belong to the study and which are outgroups, if it's the case.

The service catalogue identifies this template as **Eukaria (non-human): Variant calling, annotation and SNP-based outbreak analysis (e.g. diploid fungal outbreak)**. As part of this service, the main steps that are performed are HaplotypeCaller-based variant calling, GATK filter annotations, SNP alignment, phylogeny and coverage metrics. It does **not** include a separate functional annotation stage with VEP/SnpEff, a recombination-removal stage, or a pairwise SNP-distance matrix generator (this can be done however with MEGA).

Unlike the [Snippy service](Snippy-service.md), this workflow retains diploid genotype information. We will use SNPhylo instead. The SNPhylo script being used as part of this service represents heterozygous SNPs using IUPAC ambiguity codes instead of choosing one allele arbitrarily. The resulting tree describes relationships in the analysed SNP dataset.

### Software used

Apart from the latest version of the ``buisciii-tools``, the following software is used as part of this service:
| Component | Version or implementation in the template | Role |
|---|---|---|
| Nextflow | The template requests loading version 26.04.3 which is already available in the HPC; Sarek requires at least 25.10.4. | Runs the Sarek workflow and submits its processes to Slurm. |
| nf-core/sarek | 3.10.0 | Read preprocessing, alignment, duplicate marking, joint germline variant calling and QC. |
| GATK | 4.6.2.0 image for postprocessing | Selects variant types, adds filter labels and merges VCFs. |
| SNPhylo | [BU-ISCIII fork](https://github.com/BU-ISCIII/SNPhylo), installed at `/data/ucct/bi/pipelines/snphylo/snphylo_modified_buisciii/SNPhylo` | Converts SNP genotypes into an alignment. |
| R | The `snphylo` environment, with `getopt`, `gdsfmt` and `SNPRelate` | Executes the modified script `generate_snp_sequence.R` from SNPhylo. |
| IQ-TREE | `iqtree:3.1.4--h8471819_0` Singularity image | ModelFinder model selection and maximum-likelihood tree inference with ultrafast bootstrap. |
| Picard | `picard:3.5.0--hdfd78af_0` Singularity image | `CollectWgsMetrics` on each duplicate-marked BAM to get statistics on coverage and depth. |

Singularity and access to the Slurm partitions used in the lablogs are required.

See [Usage](Usage.md) and [Execution guidelines](Execution-guidelines.md) for the general environment and lablog conventions.

## Bioinformatics procedure

The execution order is:

```text
new-service → prepare reads → scratch (service_to_scratch)
    → Sarek → postprocessing → SNPhylo → IQ-TREE
             ↘ Picard coverage metrics, once the BAMs are complete
    → review → prepare RESULTS → finish → bioinfo-doc → remove_scratch
```

> [!IMPORTANT]
> A `lablog` script **generates scripts**. Running it does not mean that the analysis has run. Execute each generated script from the directory containing its lablog. In this template, several scripts launch `srun` commands with `&` and return before their jobs finish, but that's because they are running in their corresponding node. Do not start the next step until all jobs have been run sequentially and completed successfully!

Use Slurm and the log files to check completion:

```bash
squeue -u "$USER"
# Replace JOB_ID with the actual job ID; absence from squeue alone is not proof of success.
sacct -j JOB_ID --format=JobID,JobName,State,ExitCode
```

Commands below use `SRVCNMXXX.X` for the **resolution ID**, `SERVICE_FOLDER` for the full service directory name, and `YYYYMMDD` for the date added to the analysis directory. Replace these placeholders with the real values. The examples use `CNM/mycology`; but use the service path assigned to your request when it differs.

### 1. Create the service

Check the request and its resolution in iSkyLIMS, following [Service management](Service-management.md) and [How to manage services in iSkyLIMS](How-to-manage-services-in-iSkyLims.md). Confirm that the diploid outbreak service and the intended samples are associated with the resolution.

Activate the configured `buisciii-tools` environment and check the installed version, which should be the latest:

```bash
buisciii --version
```

From the appropriate service area, run:

```bash
cd /data/ucct/bi/services_and_colaborations/CNM/mycology
buisciii new-service SRVCNMXXX.X
```

Answer **no** to skipping folder creation for a new service (unless this is not the first resolution of the service). Reuse an existing folder only when it belongs to the intended service/resolution. `new-service` obtains the requested services from the iSkyLIMS API and copies their associated templates.

Check the resulting `SERVICE_FOLDER`. The service structure includes `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`; the diploid template itself supplies files under `ANALYSIS`, `DOC` and `RESULTS`.

```text
SERVICE_FOLDER/
├── ANALYSIS/
│   ├── samples_id.txt
│   ├── lablog_diploid_outbreak
│   └── ANALYSIS01_DIPLOID_OUTBREAK/
│       ├── lablog
│       ├── 02-postprocessing/lablog
│       ├── 03-snphylo/lablog
│       ├── 04-iqtree/lablog
│       └── 99-stats/lablog
├── DOC/
│   ├── hpc_slurm_sarek.config
│   └── sarek_params.yaml
├── RAW/
├── REFERENCES/
├── RESULTS/
│   └── lablog_diploid_outbreak_results
└── TMP/
```

`ANALYSIS/samples_id.txt` contains the service sample identifiers, one per line. Check that they match the intended isolates and the FASTQ names. Keep the buisciii execution logs under `DOC` and record any manual corrections in the service documentation.

### 2. Prepare the reads and copy the service to scratch

From the new service directory:

```bash
cd SERVICE_FOLDER/ANALYSIS
bash lablog_diploid_outbreak
```

This creates `00-reads/` with links named `SAMPLE_R1.fastq.gz` and `SAMPLE_R2.fastq.gz`, then renames the analysis directory to `YYYYMMDD_ANALYSIS01_DIPLOID_OUTBREAK` using the execution date. The lablog looks for `RAW/SAMPLE_*R1*.fastq.gz` and the corresponding R2 pattern.

Confirm that the links resolve and that every sample has both mates:

```bash
ls -l 00-reads
find 00-reads -xtype l
```

The second command should produce no broken-link paths. The preparation lablog is intended for the initial setup: rerunning its final `mv` after the analysis directory has already been renamed will fail.

Copy the service to scratch:

```bash
buisciii scratch SRVCNMXXX.X --direction service_to_scratch
```

Then enter the scratch copy:

```bash
cd /data/ucct/bi/scratch_tmp/bi/SERVICE_FOLDER/ANALYSIS/YYYYMMDD_ANALYSIS01_DIPLOID_OUTBREAK
```

On the compute nodes, this location is exposed as `/scratch/bi/SERVICE_FOLDER/...`. The lablogs translate the shared `/data/ucct/bi/scratch_tmp/` prefix into `/scratch/`. Generate analysis scripts in the scratch copy so that their `--chdir` paths point to the execution directory.

### 3. Run Sarek

#### Configure the reference and execution environment

Edit the `reference` variable in the analysis-level `lablog`; it is empty in the template:

```bash
reference="/absolute/shared/path/to/reference.fasta"
```

Choose the reference assembly for the organism, as indicated by the researcher, and record its accession/version. The reference must be accessible from the compute nodes. Use the **same reference** later in `99-stats/lablog`.

Load the HPC Nextflow 26.04.3 installation requested by the template and Singularity. Check the installed module name with the cluster's module system; do not assume that an unversioned module selects the required release.

```bash
module avail Nextflow
# Load the module corresponding to Nextflow 26.04.3.
module load singularity
nextflow -version
```

Review the two files under `DOC`:

| File | What to check |
|---|---|
| `hpc_slurm_sarek.config` | Slurm executor, `middle_idx` queue, resource limits, Singularity settings and process error handling. `params.genome = null` disables the default human genome configuration. |
| `sarek_params.yaml` | Trimming, BAM output, joint germline calling, VCF concatenation, reference-output retention and paired-calling behaviour. |

The distributed YAML contains:

```yaml
igenomes_ignore: true
trim_fastq: true
save_output_as_bam: true
joint_germline: true
concatenate_vcfs: true
save_reference: true
only_paired_variant_calling: false
```

The generated command selects `haplotypecaller`, `bwa-mem`, Spark duplicate marking and `--skip_tools baserecalibrator`. ``joint_germline`` calling produces a joint VCF, which will be then processed.

#### Generate and submit the job

From `YYYYMMDD_ANALYSIS01_DIPLOID_OUTBREAK`:

```bash
bash lablog
```

This creates links to `00-reads` and `samples_id.txt`, writes `samplesheet.csv`, and generates `sarek.sbatch` plus `_01_run_sarek.sh`.

The samplesheet has one row per sample:

```csv
patient,sample,sex,status,lane,fastq_1,fastq_2
SAMPLE01,SAMPLE01,NA,0,1,00-reads/SAMPLE01_R1.fastq.gz,00-reads/SAMPLE01_R2.fastq.gz
```

Here, each isolate is assigned its own `patient` and `sample` ID, `sex=NA`, and `status=0` for germline calling. The template represents one prepared read pair per sample with `lane=1`. These columns should not be interpreted as clinical sex or tumour information for fungal isolates.

Review the generated script, reference, samplesheet and resource requests, then submit:

```bash
bash _01_run_sarek.sh
```

The launcher submits `sarek.sbatch` using `sbatch`; Nextflow subsequently schedules the individual pipeline processes. The controller requests 4 CPUs, 12 GB and two days. Its requests are separate from those of the pipeline processes in `hpc_slurm_sarek.config`.

Wait for Sarek to finish. Inspect the dated `*_sarek.log`, `.nextflow.log` and outputs before continuing. The supplied configuration can **ignore some process exit statuses**, so do not rely only on the controller job's final status.

>[!Note]
>The **VCFTOOLS_TSTV_COUNT** process from nf-core/sarek might fail. If you check the corresponding work subfolder, you'll see:
>```
>munmap_chunk(): invalid pointer
>```
>This error is consistent with a known memory-management bug reported in VCFtools 0.1.17 when running --TsTv-by-count, including on joint-genotyped VCFs. It does not, by itself, indicate a corrupt VCF or insufficient allocated memory ([source](https://github.com/vcftools/vcftools/issues/196)).
>
>This process only calculates the transition/transversion ratio grouped by alternate allele count. **Its failure does not affect the upstream variant calling or recalibration results, and Sarek can finish with this error ignored.**
>
>If this is the only failed process, there is no need to rerun variant calling. Treat the corresponding QC metric as unavailable, even if a .TsTv.count file exists, as its contents may be incomplete or unreliable. If needed, overall Ts/Tv statistics can be obtained from bcftools stats, although these do not directly replace the allele-count-stratified report.

#### Relevant Sarek outputs

All paths in this table are relative to the dated analysis directory:

| Output | Meaning and subsequent use |
|---|---|
| `samplesheet.csv` | The input samples and read paths used by Sarek. |
| `sarek.sbatch`, `*_sarek.log`, `.nextflow.log` | Execution command and workflow diagnostics. |
| `01-sarek/preprocessing/markduplicates/SAMPLE/SAMPLE.md.bam` | Duplicate-marked alignments used by Picard in `99-stats`; duplicates are marked, not necessarily removed from the file. |
| `01-sarek/variant_calling/haplotypecaller/joint_variant_calling/joint_germline.vcf.gz` | Compressed, jointly genotyped variants; input to postprocessing. |
| `01-sarek/multiqc/multiqc_report.html` | Aggregated QC report for read processing, alignment and the tools executed by Sarek. |
| `01-sarek/multiqc/multiqc_data/` | Machine-readable data underlying the MultiQC report. |
| `01-sarek/reference/` | Reference-related files retained by `save_reference`. |
| `01-sarek/pipeline_info/` | Workflow execution/provenance reports produced by Sarek; retain the available software-version and execution records. |
| `work/` and `.nextflow/` | Intermediate tasks and execution cache needed for normal `-resume` behaviour. |

Confirm that the VCF exists and that all expected BAMs were produced. Sarek uses read-group sample names of the form `patient_sample`; with the samplesheet above, downstream VCF/alignment/tree labels may be **`SAMPLE01_SAMPLE01`**.

The stage folders and output conventions are described in [Sarek 3.10.0 documentation](https://nf-co.re/sarek/3.10.0/docs/output).

### 4. Postprocess the joint VCF

Enter the postprocessing folder and generate the scripts:

```bash
cd 02-postprocessing
module load singularity
bash lablog
```

Run the following scripts **one at a time**, checking Slurm and the corresponding logs after each stage. They contain background `srun` commands, so running all four launchers consecutively without checking completion can start jobs before their inputs exist. Therefore, wait until the previous job has successfully finished before running the next one.

| Order | Script | Action | Outputs |
|---|---|---|---|
| 1 | `_01_separate_snps_indels.sh` | GATK `SelectVariants` splits the joint VCF by variant type. | `snps.vcf.gz`, `indels.vcf.gz` and their indexes. |
| 2 | `_02_filter.sh` | GATK `VariantFiltration` adds the configured failure labels. | `snps_filtered.vcf.gz`, `indels_filtered.vcf.gz` and their indexes. |
| 3 | `_03_merge_vcfs.sh` | GATK `MergeVcfs` combines the two labelled variant sets. | `variants_fil.vcf.gz` and its index. |
| 4 | `_04_gzip.sh` | `gzip -d` decompresses the merged VCF. | `variants_fil.vcf`, the input to SNPhylo. |

For example, begin with:

```bash
bash _01_separate_snps_indels.sh
```

After **both** selection jobs finish successfully, run:

```bash
bash _02_filter.sh
```

After **both** filtering jobs finish successfully, run:

```bash
bash _03_merge_vcfs.sh
```

After merging completes successfully, run:

```bash
bash _04_gzip.sh
```

Wait for decompression to complete before starting SNPhylo. The template uses `gzip -d`, which removes the `.gz` file after successful decompression.

#### What do the filter labels mean?

A condition in the following table is a reason to **label a variant as failing**:

| Annotation | SNP failure condition | Indel failure condition | Interpretation |
|---|---|---|---|
| `QD` | `< 2.0` | `< 2.0` | Variant confidence normalised by depth. |
| `QUAL` | `< 30.0` | `< 30.0` | Site-level variant confidence. |
| `SOR` | `> 3.0` | Not configured | Strand-bias statistic. |
| `FS` | `> 60.0` | `> 200.0` | Phred-scaled Fisher strand-bias statistic. |
| `MQ` | `< 40.0` | Not configured | Root-mean-square mapping quality. |
| `MQRankSum` | `< -12.5` | Not configured | Comparison of mapping qualities for alternate- and reference-supporting reads. |
| `ReadPosRankSum` | `< -8.0` | `< -20.0` | Comparison of alternate/reference allele positions within reads. |

For example, a SNP with `QD < 2` receives `QD2` in the VCF `FILTER` column. Multiple failure labels can be present. `PASS` means the record passed the applied filters; `.` is not the same as an explicit `PASS` label.

> [!IMPORTANT]
> `VariantFiltration` adds labels; it does not remove the failing records. The template passes the merged `variants_fil.vcf` to SNPhylo **without an explicit PASS-only selection**. Do not describe this file as containing only high-quality or PASS variants.

### 5. Generate the SNP alignment with SNPhylo

After postprocessing finishes:

```bash
cd ../03-snphylo
micromamba activate snphylo
bash lablog
bash _01_snphylo.sh
```

The lablog invokes the script from the **BU-ISCIII fork**, not the upstream `snphylo.sh` wrapper. Its analysis command is:

```bash
Rscript --vanilla \
    /data/ucct/bi/pipelines/snphylo/snphylo_modified_buisciii/SNPhylo/scripts/generate_snp_sequence.R \
    -o snphylo.output -l 2 -m 0.05 -M 0.5 -t 4 \
    -v ../02-postprocessing/variants_fil.vcf
```

| Option | Meaning in this analysis |
|---|---|
| `--vanilla` | R starts without restoring saved workspaces or reading user/site startup profiles and `.Renviron` files. It is an R option, not a variant filter. |
| `-v` | Input VCF. |
| `-o snphylo.output` | Prefix for generated files. |
| `-l 2` | LD threshold; this value effectively disables correlation-based LD pruning. It does not disable the other SNP filters. |
| `-m 0.05` | Minimum minor allele frequency of 5%. Rare variants below this threshold are excluded. |
| `-M 0.5` | Maximum allowed missing-genotype proportion per SNP, calculated across samples. SNPs with missing genotypes in more than 50% of samples are excluded from downstream phylogenetic analysis. For example, with 10 samples, a SNP must have genotype calls in at least 5 samples to pass this filter, although it may still be removed by other filters. This option filters SNP sites; it does not exclude entire samples based on their overall missingness. |
| `-t 4` | Four threads for the SNPRelate operation, matching the requested CPUs. |

>[!Important]
>The values for these parameters have been internally discussed and have been selected according to literature.

The fork imports the VCF with `method="biallelic.only"`, selects SNPs using SNPRelate and writes an alignment. Its pruning call uses `autosome.only = FALSE`, allowing non-human contigs to be considered. No human chromosome limit is supplied in this template. Heterozygous genotypes are represented with IUPAC codes, for example `R` for A/G and `Y` for C/T; missing genotypes are written as `N`.

This stage has its own biallelic-SNP and MAF/missingness selection. That should not be confused with enforcing the GATK `FILTER=PASS` labels from the preceding stage. See the [fork's R script](https://github.com/BU-ISCIII/SNPhylo/blob/master/scripts/generate_snp_sequence.R) for the implementation used by the shared installation.

| Output | Meaning |
|---|---|
| `snphylo.output.gds` | Genotype data imported into GDS; the alignment is extracted from a selected set of sites, so the GDS file is not itself the final alignment. |
| `snphylo.output.fasta` | Multi-FASTA SNP alignment used by IQ-TREE, with one sequence per sample. This file will be used for the construction of the corresponding phylogenetic tree. |
| `snphylo.output.id.txt` | Identifiers of selected SNPs from the GDS `snp.rs.id` field. IDs can be missing or non-unique; this is not necessarily a chromosome-position table. |
| `logs/SNPHYLO.<jobid>.log` | Import, filtering, retained-SNP and execution messages. |

Check that the alignment is nonempty, contains the expected samples and has equal-length sequences. At least four usable sequences are needed for the IQ-TREE analysis described here; inspect logs for samples with excessive missing data or too few informative sites. The template does not generate a pairwise SNP-distance matrix.

### 6. Infer the phylogenetic tree with IQ-TREE

This template uses **two executions of the IQ-TREE lablog**. The second section is initially commented out.

#### 6.1 Select the substitution model

After SNPhylo completes:

```bash
cd ../04-iqtree
module load singularity
bash lablog
bash _00_iqtreemfp.sh
```

The active `#1` section creates `logs/` and `_00_iqtreemfp.sh`, which runs IQ-TREE with `-m MFP` on `../03-snphylo/snphylo.output.fasta`. This performs ModelFinder model selection. The script backgrounds the Slurm command: wait for `IQTREEMFP` to complete and inspect `logs/IQTREEMFP.<jobid>.log`.

>[!Tip]
>For more information on IQTREE's substitution models, please visit: https://iqtree.github.io/doc/Substitution-Models.

Locate the selected model:

```bash
grep 'Best' logs/IQTREEMFP*
```

Check the model against the IQ-TREE report, rather than assuming it is the same as in another service. When no output prefix is specified, IQ-TREE normally places its result files next to the input alignment, for example `03-snphylo/snphylo.output.fasta.iqtree`.

#### 6.2 Run the final tree and bootstrap analysis

Edit `04-iqtree/lablog`:

1. Comment out the active commands under `#1`, including `mkdir logs` and the `echo` that generates `_00_iqtreemfp.sh`.
2. Uncomment the two commands under `#2`: the `bestfitmodel=...` assignment and the `echo` that generates `_01_iqtreeall.sh`.
3. Keep the `scratch_dir` assignment active.

The model assignment searches **all** matching `logs/IQTREEMFP*` files. If there are logs from multiple attempts, restrict it to the successful current run or otherwise select that run's model explicitly. Confirm that exactly one valid model is inserted into the generated command.

Generate and run the second script:

```bash
bash lablog
cat _01_iqtreeall.sh
bash _01_iqtreeall.sh
```

This runs IQ-TREE with the selected model, `-T 20`, `-B 1000` and `-pre phylo.iqtree.bootstrap`. The 1000 replicates are **ultrafast bootstrap** replicates. Again, wait for the `IQTREEFULLALIGN` job to finish and review its log.

| Output in `04-iqtree/` | Meaning |
|---|---|
| `phylo.iqtree.bootstrap.treefile` | Inferred maximum-likelihood tree in Newick format, including branch support from the analysis. This is the tree linked into the delivery. This file can be loaded into [**iTOL**](https://itol.embl.de/) for visualization and image generation. |
| `phylo.iqtree.bootstrap.iqtree` | Human-readable report: model, likelihood, alignment information and analysis details. |
| `phylo.iqtree.bootstrap.log` | IQ-TREE's own execution log. |
| `phylo.iqtree.bootstrap.contree` | Bootstrap consensus tree when produced by the run; distinct from the ML `.treefile`. |
| `phylo.iqtree.bootstrap.ckp.gz` | Checkpoint used for restarting the same IQ-TREE analysis; not a researcher-facing result. |
| `logs/IQTREEFULLALIGN.<jobid>.log` | Slurm-captured output for the final analysis. |

The template does **not** enable ascertainment correction (`+ASC`). There is a comment in the template about ASC, indicating that performing ascertainment bias correction is not possible since the alignment from SNPhylo contains invariable positions. Do not add ASC without examining constant or constant-compatible columns, including the effects of missing data and IUPAC ambiguity. A SNP-only alignment is not a whole-genome alignment, and its branch lengths should not be reported directly as substitutions per base of the entire genome. See the [IQ-TREE command reference](https://iqtree.github.io/doc/Command-Reference).

### 7. Collect whole-genome coverage metrics

`99-stats` uses **Picard CollectWgsMetrics**, independently of the phylogenetic inference. It can run once Sarek's duplicate-marked BAMs are complete; it does not need to wait for SNPhylo or IQ-TREE. Finish this QC review before delivery.

From the dated analysis directory:

```bash
cd /data/ucct/bi/scratch_tmp/bi/SERVICE_FOLDER/ANALYSIS/YYYYMMDD_ANALYSIS01_DIPLOID_OUTBREAK/99-stats
module load singularity
```

Edit `reference` in this folder's `lablog` to match **exactly the FASTA used by Sarek**. Then generate and run:

```bash
bash lablog
bash _01_wgsmetrics.sh
```

`_01_wgsmetrics.sh` launches one background `srun` per sample, taking `../01-sarek/preprocessing/markduplicates/SAMPLE/SAMPLE.md.bam` as input.

Wait until **all** Picard jobs have completed successfully and check that every sample has `picard/SAMPLE_collect_wgs_metrics.txt`. Only then run:

```bash
bash _02_gather_wgs_metrics.sh
```

The collection script finds the `GENOME_TERRITORY` header, reads the following data row for each sample and adds `SAMPLENAME`. It then selects columns `1,2,3,14,17` from the combined table.

| Output | Meaning |
|---|---|
| `picard/SAMPLE_collect_wgs_metrics.txt` | Per-sample Picard WGS metrics and histogram information. |
| `logs/PICARD_SAMPLE.<jobid>.log` | Per-sample execution messages. |
| `wgs_metrics_all.txt` | Combined metrics table containing all Picard metric columns and the sample name. |
| `wgs_metrics_all_filtered.txt` | Five selected columns for review; “filtered” refers to **column selection**, not removal of low-quality samples. |

The expected selected columns, including the added `SAMPLENAME`, are:

| Column | Metric | Interpretation |
|---|---|---|
| 1 | `SAMPLENAME` | Service sample identifier. |
| 2 | `GENOME_TERRITORY` | Number of non-N reference bases over which coverage is evaluated. |
| 3 | `MEAN_COVERAGE` | Mean coverage after the tool's filters. |
| 14 | `PCT_EXC_TOTAL` | Fraction of aligned bases excluded by all applicable filters. |
| 17 | `PCT_10X` | Fraction of the genome territory covered at least 10 times after filtering. |

Picard's `PCT_*` values are fractions, so `0.95` means 95%. Take into account the tool's defaults: it uses minimum mapping/base qualities of 20, a coverage cap of 250 and does not count unpaired reads by default. These settings are not overridden in this template. See the [Picard 3.5.0 implementation](https://github.com/broadinstitute/picard/blob/3.5.0/src/main/java/picard/analysis/CollectWgsMetrics.java) and [WgsMetrics definitions](https://github.com/broadinstitute/picard/blob/3.5.0/src/main/java/picard/analysis/WgsMetrics.java).

## Review the results

Review the results before preparing the delivery:

- Check read and alignment QC in `01-sarek/multiqc/multiqc_report.html`, then examine the Picard coverage metrics for samples that may need investigation or resequencing.
- Confirm that the VCF, SNP alignment and tree contain the expected isolates, accounting for Sarek's `patient_sample` labels.
- Inspect the number of retained SNPs and missing/ambiguous data. The MAF threshold can remove rare differences, and missingness can affect which sites enter the analysis.
- Open the final `.treefile` in iTOL, click on "Midpoint root" in Advanced Settings and add bootstrap values, modify node symbols, customize sample names and export the final tree in different formats (usually SVG, PDF, NWK and PNG).

If a sample needs to be excluded or another reference is required, retain the original analysis and rerun in a different analysis folder. Removing a label from the final tree is not equivalent to recomputing an alignment and phylogeny!

## Prepare the delivery

From the service's `RESULTS` directory in scratch:

```bash
cd /data/ucct/bi/scratch_tmp/bi/SERVICE_FOLDER/RESULTS
bash lablog_diploid_outbreak_results
```

The lablog creates:

```text
RESULTS/YYYYMMDD_entrega01/variant_calling/
├── multiqc_report.html
├── variants_fil.vcf
└── phylo.iqtree.bootstrap.treefile
```

These are **symbolic links** to the corresponding analysis files. The delivery date is the date on which the results lablog runs; it does not need to match the analysis date.

The distributed lablog searches `ANALYSIS/*_ANALYSIS01_DIPLOID_OUTBREAK`. If there are multiple matching analyses, or the analysis number/name has changed, edit those links to point explicitly to the intended analysis before execution. Check for broken links:

```bash
find YYYYMMDD_entrega01 -xtype l
```

The default results lablog links exactly the three files above. It does **not** add the SNP alignment, IQ-TREE report or Picard tables. If the agreed delivery includes them, add those files explicitly and record the additions. For example, from the service's `RESULTS` folder, after replacing both dates with the actual ones:

```bash
ln -s ../../../ANALYSIS/ANALYSIS_DATE_ANALYSIS01_DIPLOID_OUTBREAK/99-stats/wgs_metrics_all_filtered.txt \
    YYYYMMDD_entrega01/variant_calling/
ln -s ../../../ANALYSIS/ANALYSIS_DATE_ANALYSIS01_DIPLOID_OUTBREAK/03-snphylo/snphylo.output.fasta \
    YYYYMMDD_entrega01/variant_calling/
ln -s ../../../ANALYSIS/ANALYSIS_DATE_ANALYSIS01_DIPLOID_OUTBREAK/04-iqtree/phylo.iqtree.bootstrap.iqtree \
    YYYYMMDD_entrega01/variant_calling/
```

## Complete the service with ``buisciii-tools``

### Store and transfer the reviewed results

Once the analysis, QC and delivery contents have been reviewed, run:

```bash
buisciii finish SRVCNMXXX.X
```

Select/check the SFTP destination for this researcher. `finish` performs service cleanup, transfers scratch back to the service area, applies service-directory renaming and copies the service to SFTP. Review its logs and the destination contents.

The `diploid_outbreak` entry in [services.json](https://github.com/BU-ISCIII/buisciii-tools/blob/2.3.5/buisciii/templates/services.json) has empty service-specific `clean.folders` and `clean.files` lists. `RAW` and `TMP` are marked as `no_copy`, and `RESULTS` is the configured final-results folder. Review the actual transfer and cleanup behaviour before deleting anything needed to reproduce or deliver the analysis.

### Generate the service and delivery documentation

Use the environment/workstation with access to the configured `bioinfo_doc` storage. Generate the service information and deliver the service after storing the delivery and email notes in separate .txt files:

```bash
buisciii bioinfo-doc SRVCNMXXX.X --type service_info
```



```bash
buisciii bioinfo-doc SRVCNMXXX.X --type delivery
```

The delivery workflow records delivery information and can send the notification email. Check the generated PDF, researcher destination, delivery notes and email content before confirming delivery actions.

### Remove the scratch copy

After confirming that the service is stored correctly and the delivery mail has been sent, and the scratch copy is no longer needed, please run:

```bash
buisciii scratch SRVCNMXXX.X --direction remove_scratch
```

This is a deletion step, not a prerequisite for generating results. Keep the service records and delivery documentation according to the unit's procedures.

## Diploid Outbreak report template

Use the following outline for the internal results review and adapt it into the delivery report. Fill it with the actual run information; it is not an automatically generated result.

```text
Servicios: nf-core/sarek (haplotypecaller) + SNPhylo y IQTREE
Número de muestras: XX
Solicitante: XXX
Organismo indicado: XXX XXX
Estado: Finalizado y copiado al SFTP
Cantidad de lecturas: X.XM - XXM
Calidad general: XXX
Incidencias o notas --> anything that both the Unit members or the researcher should know about.
```
