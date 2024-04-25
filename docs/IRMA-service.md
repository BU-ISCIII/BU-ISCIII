# IRMA

IRMA does not include quality trimming of the reads by itself. Therefore, before the execution of the pipeline the samples are analysed with FastQC for quality control and pre-processed with fastp.

## FASTQC

[FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) gives general quality metrics about your sequenced reads. It provides information about the quality score distribution across your reads, per base sequence content (%A/T/G/C), adapter contamination and overrepresented sequences. For further reading and documentation see the [FastQC help pages](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/).

<details markdown="1">
<summary>Output files</summary>

- `fastqc/raw/`
  - `*_fastqc.html`: FastQC report containing quality metrics.
  - `*_fastqc.zip`: Zip archive containing the FastQC report, tab-delimited data file and plot images.

**NB:** The FastQC plots in this directory are generated relative to the raw, input reads. They may contain adapter sequence and regions of low quality. To see how your reads look after trimming please refer to the FastQC reports in the `fastqc/trim/` directory.

</details>

![](images/mqc_fastqc_plot.png)

## FASTP

[Fastp](https://github.com/OpenGene/fastp?tab=readme-ov-file#fastp) is a tool designed to provide fast, all-in-one preprocessing for FastQ files. It has been developed in C++ with multithreading support to achieve higher performance. This tools is used to pre-process the reads using multiple filters. For this specific workflow it is used for quality trimming and to filter any short-reads, adapters, polyG and polyX. This software includes in its output an html with several stats that show the state of the reads before and after processing.

<details markdown="1">
<summary>Output files</summary>

- `fastp/`
  - `*.fastp.html`: Trimming report in html format.
  - `*.fastp.json`: Trimming report in json format.
- `fastp/log/`
  - `*.fastp.log`: Trimming log file.
- `fastqc/trim/`
  - `*_fastqc.html`: FastQC report of the trimmed reads.
  - `*_fastqc.zip`: Zip archive containing the FastQC report, tab-delimited data file and plot images.

</details>

![](images/mqc_fastp_plot.png)

## IRMA workflow

[IRMA (Iterative Refinement Meta-Assembler)](https://wonder.cdc.gov/amd/flu/irma/) was designed for the robust assembly, variant calling, and phasing of highly variable RNA viruses. IRMA is deployed for this service to analyse Influenza virus. IRMA is free to use and parallelizes computations for both cluster computing and single computer multi-core setups. You can read the [IRMA manuscript](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-016-3030-6#Sec9) for more background on the methodology. 

![](images/IRMA_workflow.png)

**IRMA** uses several auxiliar softwares in its workflow:

- [BLAT](http://www.kentinformatics.com/products.html) for the match step
- [LABEL](https://wonder.cdc.gov/amd/flu/label), which also packages certain resources used by IRMA:
  - [Sequence Alignment and Modeling System (SAM)](http://www.ncbi.nlm.nih.gov/pubmed/9927713) for both the rough align and sort steps
  - [Shogun Toolbox](https://github.com/shogun-toolbox/shogun), which is an essential part of LABEL, is used in the sort step.
- [SSW](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0082138) for the final assembly step.
- [MINIMAP2](https://academic.oup.com/bioinformatics/article/34/18/3094/4994778) for the final assembly step (alternate option that may be useful for long - read assemblies).
- [samtools](http://samtools.sourceforge.net/) for BAM-SAM conversion as well as BAM sorting and indexing
- [GNU Parallel](http://www.gnu.org/software/parallel/) for single node parallelization

### IRMA output directory structure

**_gene\_segment_** in the case of influenza can be any of the following: [HA, NA, NS, MP, NP, PA, PB1, PB2] and HE for Influenza-C

### **Final files:**

- **_gene\_segment_.bam**			Sorted BAM file for the final gene_segment assembly (merged if applicable)
- **_gene\_segment_.bam.bai**			BAM file index for gene_segment assembly
- **_gene\_segment_.fasta**			Final assembled plurality consensus (no mixed base calls) for gene_segment
- **_gene\_segment_.a2m**			Optional file: Plurality consensus aligned to profile HMM
- **_gene\_segment_.vcf**			Custom variant call file for called IRMA variants for each gene segment

### **Folders:**

<u>**amended_consensus/</u>**: Assembled consensus per gene segment with missed based calls. Numbers correspond to fragments in $SEG_NUMBERS
- **_sample\_name_\__fragment\_number_.fa**		Amended consensus
- **_sample\_name_\_7.a2m**		Optional output: amended global alignment to profile HMM
- **_sample\_name_\_7.pad.fa**		Optional output: N-padded consensus for amplicon dropouts

<u>**figures/**</u>

  - **_gene\_segment_-coverageDiagram.pdf**		Shows coverage and variant calls
  - **_gene\_segment_-heuristics.pdf**		Heuristic graphs for gene segment
  - **_gene\_segment_-EXPENRD.pdf**		gene segment variant phasing using experimental enrichment distances
  - **_gene\_segment_-JACCARD.pdf**		gene segment variant phasing using modified Jaccard distances
  - **_gene\_segment_-MUTUALD.pdf**		gene segment variant phasing using mutual association distances
  - **_gene\_segment_-NJOINTP.pdf**		gene segment variant phasing using normalized joint probability distances
  - **READ_PERCENTAGES.pdf**		Break down for reads assembled

<u>**intermediate/**</u> Intermediate data for each step

  - <u>**0-ITERATIVE-REFERENCES/**</u>
    - **R0-_gene\_segment_.ref**	Starting reference library sequence for segment
    - **R1-_gene\_segment_.ref**	Gene segment working reference after round 1,template for round 2
    - **R2-_gene\_segment_.ref**	Working reference for gene segment after round 2
  - <u>**1-MATCH_BLAT/**</u>		
    - **R1.tar.gz**	
    - **R2.tar.gz**	Archive of BLAT results for the MATCH step
    - **R3.tar.gz**	
  - <u>**2-SORT_BLAT/**</u>		
    - **R1.tar.gz**	Classification/sorting intermediate files for round 1
    - **R1.txt**	Summary statistics of sorting results for round 1
    - **R2.tar.gz**	Classification/sorting intermediate files for round 2
    - **R2.txt**	Summary statistics of sorting results for round 2
  - <u>**3-ALIGN_SAM/BLAT/**</u>		
    - **storedCounts.tar.gz**	Statistic files used to create rough consensus sequences
  - <u>**4-ASSEMBLE_SSW/**</u>	Final assembly phase intermediate iterativeresuls
    - **F1-_gene\_segment_.bam**	Unsorted BAM file for gene segment assembly iteration 1
    - **F1-_gene\_segment_.ref**	Reference for final assembly, gene segment,   iteration 1
    - **F2-_gene\_segment_.bam**	Unsorted BAM file for gene segment assembly,   iteration 2
    - **F2-_gene\_segment_.ref**	Reference for final assembly, gene segment,   iteration 2
    - **reads.tar.gz**	Archive of sorted, unmerged reads by gene segment

<u>**logs/**</u>	

  - **ASSEMBLY_log.txt**		SSw scores per all rounds tried in the iterative refinement
  - **NR_COUNTS_log.txt**		Read pattern counts at various stages
  - **QC_log.txt**		Quality control output
  - **READ_log.txt**		Counts of assembled reads from BAM files
  - **FLU-_sample\_name_.sh**		Configuration file corresponding to this IRMA run
  - **run_info.txt**		Table of parameters used by the IRMA run

<u>**matrices/**</u>			Phasing matrices used to generate heat maps

  - **_gene\_segment_-EXPENRD.sqm**	
  - **_gene\_segment_-JACCARD.sqm**		
  - **_gene\_segment_-MUTUALD.sqm**		
  - **_gene\_segment_-NJOINTP.sqm**	

<u>**secondary/**</u>

  - **R1-A_NA_N1.fa**		Trace A_NA_N1 sorted into secondary status
  - **R1-UNRECOGNIZABLE.fa**		Read patterns that matched flu but had poor signal according to LABEL
  - **R2-UNRECOGNIZABLE.fa**		
  - **unmatched_read_patterns.tar.gz**		Archive of left over read patterns that did not match FLU

<u>**tables/**</u>			
  - **_gene\_segment_-pairingStats.txt**		Summary of paired-end merging statistics, if applicable, gene segment
  - **_gene\_segment_-coverage.txt**		Summary coverage statistics for assembly, gene segment
  - **_gene\_segment_-coverage.a2m.txt**		Optional file: Coverage statistics for plurality consensus globally aligned to profile HMM
  - **_gene\_segment_-coverage.pad.txt**		Optional file: Coverage statistics for padded plurality consensus globally aligned to profile HMM
  - **_gene\_segment_-allAlleles.txt**		Statistics for every position & allele in the assembly, gene segment
  - **_gene\_segment_-insertions.txt**		Called insertion variants for gene segment
  - **_gene\_segment_-deletions.txt**		Called deletion variants for gene segment
  - **_gene\_segment_-variants.txt**		Called single nucleotide variants for gene segment
  - **READ_COUNTS.txt**		Read counts for various points in the assembly process