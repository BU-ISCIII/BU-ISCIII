# Introduction

BU-ISCIII is focus on the analysis of high throughput data (NGS) inside Computational and functional genomics. There are three service categories:

- **Genomic data analysis:** bioinformatic analysis for different kind of data and experiments with fixed input and output files.
- **Bioinformatics counseling:** bioinformatic consulting for experiment and analysis design. Also if your research interest does not fit any of our fixed service offering ask of this service and we will evaluate a possible collaboration. Moreover we offer support for training: courses organization, internship, MSc/Phd thesis,...
- **User support:** we support installation of software in linux machines, and we offer the deployment of custom virtual machines in our server Bioinfo01 for researchers interested in performing their own analysis. Moreover we offer the possibility of develop small code snippets for specific functionality the researcher may require.

## Service portfolio

Genomic Data Analysis:

- Pre-processing and quality analysis
- Sequence quality analysis and host genome removal ([seek_and_destroy](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/seek_and_destroy))
- Next Generation Sequencing data analysis
- DNAseq / cDNAseq: Exome sequencing (WES) / Genome sequencing (WGS) / Targeted sequencing
  - Low-frequency variants detection and annotation for whole genome or sequencing panel (e.g. retinoblastoma gene panel) ([lowfreq_panel](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/lowfreq_panel))
  - Eukaria: Variant calling and annotation for a sequencing panel (e.g. epidermolysis gene panel, mouse or rat gene panel) ([exomeeb](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/exomeeb))
  - Eukaria (non-human): Variant calling, annotation and SNP-based outbreak analysis (e.g. diploid fungal outbreak) (TODO freebayes_outbreak)
  - Human:  Exome sequencing for variant calling, annotation and inheritance filtering (e.g. Exome sequencing of a human trio (two parents and one child))  ([exometrio](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/exometrio))
  - Human: Whole genome sequencing for SNPs variant calling, annotation and  inheritance filtering (e.g.WGS of a human trio )  ([wgstrio](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/wgstrio))
  - Fungal / bacteria / virus : Variant calling, annotation and SNP-based outbreak analysis (e.g. haploid fungal outbreak) ([snippy](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/snippy))
  - Bacteria: _De novo_ genome assembly and annotation ([Assembly](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/assembly))
  - Bacteria:  In-depth analysis of Mycobacterium species genomes (e.g. _M. tuberculosis_. _M. bovis_) ([MTBSeq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/mtbseq))
  - Bacteria: Plasmid analysis and characterization ([PlasmidID](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/plasmidid))
  
    PlasmidID is a mapping-based, assembly-assisted plasmid identification tool that analyzes and gives graphic solution for plasmid identification.

    PlasmidID is a computational pipeline that maps Illumina reads over plasmid database sequences. The k-mer filtered, most covered sequences are clustered by identity to avoid redundancy and the longest are used as scaffold for plasmid reconstruction. Reads are assembled and annotated by automatic and specific annotation. All information generated from mapping, assembly, annotation and local alignment analyses is gathered and accurately represented in a circular image which allow user to determine plasmidic composition in any bacterial sample.

    Below are the files that **researchers NEED to provide** when requesting the plasmidID service.

    <details markdown="1">
    <summary>Required information for service request</summary>

    - As default annotation databases we use:
      - AMR resistance genes: Card database
      - Virulence genes: VirulenceFinder database
      - IS: NCBI sequences
      - Rep/INC genes: plasmidFinder database (Caratoli et al. 2014)
    - If you want a specific database you need to provide a multifasta with the sequence features you want to annotate, or indicate a url where we can download the resource.
    </details>

  - Bacteria: Multi-Locus Sequence Typing (MLST), analysis of virulence factors, antimicrobial resistance, and plasmids characterization ([characterization](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/characterization))
  - Bacteria: Core genome or whole genome Multi-Locus Sequence Typing analysis (cg/wgMLST) (TODO wgmlst_chewbbaca)
  - Viral: Genomic reconstruction, variant calling and _de novo_ assembly ([viralrecon](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/viralrecon))
  - Viral Flu: Influenza fragment reconstruction and variant detection ([IRMA](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/IRMA))
- RNAseq: Transcriptome sequencing  ([rnaseq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/rnaseq))
  - Differential Gene Expression (DEG) ([rnaseq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/rnaseq))
  The RNAseq service performs a quality control (QC), trimming and alignment followed by quantification with [Star](https://github.com/alexdobin/STAR) and [Salmon](https://combine-lab.github.io/salmon/), respectively.
  After quantification, differntial expression analysis is carried out with [DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html).
  Below are the files that **researchers NEED to provide** when requesting the RNA-seq service.
  <details markdown="1">
  <summary>Required information for service request</summary>
    <b>Service Notes Description</b>
    In this field (when requesting service in <i>iskylims</i>), the researcher needs to provide relevant information such as which type of NGS data is going to be analyzed. Options include:

      Genes (mRNA)
      Transcripts

    **comparatives.txt**

    The `comparatives.txt` file defines the experimental design for the analysis. It specifies the comparison order, sense, and direction between sample groups. Each comparison requested should have a corresponding line in this file. The file format consists of three columns without headings:

    1. Incremental index representing each comparison.
    2. Baseline group/s.
    3. Group against which the baseline group/s in the first column is compared.

    Example:

    ```Bash
    1 Treatment Control
    2 Treatment       Control
    3 Treatment       Control
    4 Treatment1-Treatment2       Control1-Control2
    ```

    **clinical_data.txt**

    The `clinical_data.txt` file is necessary for categorizing the names of samples into comparison groups. This file comprises two columns:

  - **Name:** Sample name.
  - **Group:** Group to which the sample belongs.

    Example:

    ```Bash
       Name    Group
    ```

  </details>

  - Differential transcript expression (DET) ([rnaseq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/rnaseq))
  The RNAseq service performs a quality control (QC), trimming and alignment followed by quantification with [Star](https://github.com/alexdobin/STAR) and [Salmon](https://combine-lab.github.io/salmon/), respectively.
  After quantification, differntial expression analysis is carried out with [fishpond](https://www.bioconductor.org/packages/release/bioc/html/fishpond.html).
  Below are the files that researchers need to provide when requesting the RNA-seq service.
  - Differential miRNA expression (DEM) (TODO mirnaseq)
  - Gene expression changes over a series of time points (TODO timeseries_rnaseq)
  The RNAseq service performs a quality control (QC), trimming and alignment followed by quantification with [Star](https://github.com/alexdobin/STAR) and [Salmon](https://combine-lab.github.io/salmon/), respectively.
  After quantification, differntial expression analysis is carried out with ad-hoc software/scripts.
  Below are the files that researchers need to provide when requesting the RNA-seq service.
- Metagenomics and targeted metagenomics
  - Taxonomic based Identification and classification of organisms in complex communities (TODO mag_met)
  - _De novo_ assembly contigs' alignment to database [BLAST](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/blast_nt)(TODO blast_nt)
  - Bacteria: 16S rRNA gene analysis to assess bacterial diversity (TODO 16s_metagenomics)
  - Viral:  Detection and characterization of viral genomes within metagenomic data ([pikavirus](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/bu_isciii/templates/pikavirus))

- **Bioinformatics consulting and training**
  - Bioinformatics analysis consulting
  - In-house and outer course organization
  - Student training in colaboration: Master thesis, research visit,...

- **User support**
  - Installation and support of bioinformatic software on Linux OS
  - Installation and access to Virtual machines in the Unit server containing bioinformatic software
  - Code snippets development
