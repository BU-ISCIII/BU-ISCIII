# Introduction

BU-ISCIII is focus on the analysis of high throughput data (NGS) inside Computational and functional genomics. There are three service categories:

- **Genomic data analysis:** bioinformatic analysis for different kind of data and experiments with fixed input and output files.
- **Bioinformatics counseling:** bioinformatic consulting for experiment and analysis design. Also if your research interest does not fit any of our fixed service offering ask of this service and we will evaluate a possible collaboration. Moreover we offer support for training: courses organization, internship, MSc/Phd thesis,...
- **User support:** we support installation of software in linux machines, and we offer the deployment of custom virtual machines in our server Bioinfo01 for researchers interested in performing their own analysis. Moreover we offer the possibility of develop small code snippets for specific functionality the researcher may require.

# Service portfolio

## **Genomic Data Analysis:**

- ### Sequence quality analysis and host genome removal ([seek_and_destroy](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/seek_and_destroy))

- ### DNAseq / cDNAseq: Exome sequencing (WES) / Genome sequencing (WGS) / Targeted sequencing

  - #### **Low-frequency variants detection and annotation for whole genome or sequencing panel (e.g. retinoblastoma gene panel) ([lowfreq_panel](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/lowfreq_panel))**

    The Low-Frequency Variants Detection and Annotation Service processes whole genome or targeted sequencing panel data (e.g., retinoblastoma gene panel) using the lowfreq_panel pipeline. The pipeline includes quality control steps with fastp and FastQC, maps reads to the reference genome with bwa mem, and detects low-frequency variants using VarScan. Finally, the detected variants are annotated with kggseq and presented as an annotated table in an Excel file.

    Below are the files that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary>

       <b>Service Notes Description</b>

       When requesting a service in iskylims, researchers must provide all necessary files and information to ensure the accurate processing of data and annotation of variants.

    - **Reference genome or sequence**: NCBI sequence ID or FASTA file of the reference genome used for read alignment and variant calling.
    - **targeted_regions.bed (optional):** a file with targeted genomic coordinates during sequencing protocol in [BED format](https://www.ensembl.org/info/website/upload/bed.html), consists of one line per feature.

         ```
         chrom - name of the chromosome or scaffold. Any valid seq_region_name can be used, and chromosome names can be given with or without the 'chr' prefix.
         chromStart - Start position of the feature in standard chromosomal coordinates (i.e. first base is 0).
         chromEnd - End position of the feature in standard chromosomal coordinates
         name - Label to be displayed under the feature. Optional.

         1       chromStart   chromEnd   orientation(+/-)       name
         1       chromStart   chromEnd   orientation(+/-)       name
        ...
        chrX       chromStart   chromEnd   orientation(+/-)       name
        ```

    </details>

  - #### **Eukaria: Variant calling and annotation for a sequencing panel (e.g. epidermolysis gene panel, mouse or rat gene panel) ([exomeeb](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/exomeeb))**

    ExomeEB service uses nextflow's pipeline sarek to detect variants on whole genome or targeted sequencing data, in this case exome for single samples. The output is then processed with GATK-toolkit and annotated with [Ensembl's Variant Effect Predictor (VEP)](https://www.ensembl.org/info/docs/tools/vep/index.html) and [Exomiser](https://exomiser.readthedocs.io/en/latest/advanced_analysis.html) which will include prediction of effect and inheritance mode, targeting a specific list of genes if given by the researcher.

    Below are the files that **researchers NEED to provide** when requesting the ExomeEB service.

    <details markdown="1">
    <summary>Required information for service request</summary>
    <b>Service Notes Description</b>

    When requesting a service in iskylims, researchers are required to provide pertinent details, including a list of targeted genes to analyse during exomiser's annotation step if necessary.

    - **targeted_regions.bed:** a file with targeted genomic coordinates during sequencing protocol in [BED format](https://www.ensembl.org/info/website/upload/bed.html), consists of one line per feature.

    ```
    chrom - name of the chromosome or scaffold. Any valid seq_region_name can be used, and chromosome names can be given with or without the 'chr' prefix.
    chromStart - Start position of the feature in standard chromosomal coordinates (i.e. first base is 0).
    chromEnd - End position of the feature in standard chromosomal coordinates
    name - Label to be displayed under the feature. Optional.

    1       chromStart   chromEnd   orientation(+/-)       name
    1       chromStart   chromEnd   orientation(+/-)       name
    ...
    chrX       chromStart   chromEnd   orientation(+/-)       name
    ```

    </details>

  - #### **Eukaria (non-human): Variant calling, annotation and SNP-based outbreak analysis (e.g. diploid fungal outbreak) ([freebayes_outbreak](https://github.com/BU-ISCIII/buisciii-tools/tree/main/bu_isciii/templates/freebayes_outbreak))**

    The Eukaria (non-human): Variant calling, annotation and SNP-based outbreak analysis service is designed for identifying variants and performing phylogenetic analysis in diploid non-eukaria genomes (e.g., diploid fungal outbreaks). This service uses the snpphylo pipeline, combining tools like nf-core/sarek for quality control, mapping with bwa mem, and variant calling using GATK or freebayes. The identified SNPs are further processed with snpPhylo to generate an SNP matrix, and IQ-TREE is used to construct the phylogenetic tree.

    Below are the files that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary>

    <b>Service Notes Description</b>

    When requesting a service in iskylims, researchers must provide essential files and information to ensure accurate variant calling and downstream phylogenetic analysis.

    - **Reference genome**: The genome assembly of the organism being analyzed, in FASTA format. This file will be used for mapping and variant calling.

    - **Sample metadata** (optional): A plain text file containing metadata for the samples (e.g., sample IDs, collection dates, or locations) to facilitate outbreak analysis and improve tree visualization.

    </details>

  - #### **Human:  Exome sequencing for variant calling, annotation and inheritance filtering (e.g. Exome sequencing of a human trio (two parents and one child))  ([exometrio](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/exometrio))**

    Exometrio service uses nextflow's pipeline sarek to detect variants on whole genome or targeted sequencing data, in this case exome for multiple related samples, ussually relatives. The output is then processed with GATK-toolkit and annotated with [Ensembl's Variant Effect Predictor (VEP)](https://www.ensembl.org/info/docs/tools/vep/index.html) and [Exomiser](https://exomiser.readthedocs.io/en/latest/advanced_analysis.html) which will include prediction of effect and inheritance mode.

    Below are the files that **researchers NEED to provide** when requesting the Exometrio service.

    <details markdown="1">
    <summary>Required information for service request</summary>

    - **targeted_regions.bed:** a file with targeted genomic coordinates during sequencing protocol in [BED format](https://www.ensembl.org/info/website/upload/bed.html), consists of one line per feature.

    ```
    chrom - name of the chromosome or scaffold. Any valid seq_region_name can be used, and chromosome names can be given with or without the 'chr' prefix.
    chromStart - Start position of the feature in standard chromosomal coordinates (i.e. first base is 0).
    chromEnd - End position of the feature in standard chromosomal coordinates
    name - Label to be displayed under the feature. Optional.

    1       chromStart   chromEnd   orientation(+/-)       name
    1       chromStart   chromEnd   orientation(+/-)       name
    ...
    chrX       chromStart   chromEnd   orientation(+/-)       name
    ```

    - **family.ped:** A pedigree file following [PED format](https://gatk.broadinstitute.org/hc/en-us/articles/360035531972-PED-Pedigree-format).

    ```
    family.ped

    group   samplefather_samplefather   0       0       1       1
    group   samplemother_samplemother   0       0       2       1
    group   samplechildren_samplechildren   samplefather_samplefather   samplemother_samplemother   1       2
    ```

    </details>

  - #### **Human: Whole genome sequencing for SNPs variant calling, annotation and  inheritance filtering (e.g.WGS of a human trio )  ([wgstrio](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/wgstrio))**

    WGStrio service uses nextflow's pipeline sarek to detect variants on whole genome or targeted sequencing data, in this case Whole genome for multiple related samples, ussually relatives. The output is then processed with GATK-toolkit and annotated with [Ensembl's Variant Effect Predictor (VEP)](https://www.ensembl.org/info/docs/tools/vep/index.html) and [Exomiser](https://exomiser.readthedocs.io/en/latest/advanced_analysis.html) which will include prediction of effect and inheritance mode.

    Below are the files that **researchers NEED to provide** when requesting the WGStrio service.

    <details markdown="1">
    <summary>Required information for service request</summary>

    - **family.ped:** A pedigree file following [PED format](https://gatk.broadinstitute.org/hc/en-us/articles/360035531972-PED-Pedigree-format).

    ```
    family.ped

    group   samplefather_samplefather   0       0       1       1
    group   samplemother_samplemother   0       0       2       1
    group   samplechildren_samplechildren   samplefather_samplefather   samplemother_samplemother   1       2
    ```

    </details>

  - #### **Fungal / bacteria / virus : Variant calling, annotation and SNP-based outbreak analysis (e.g. haploid fungal outbreak) ([snippy](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/snippy))**
  
    The Fungal/Bacterial/Viral Variant Calling, Annotation, and SNP-Based Outbreak Analysis Service is tailored for identifying variants and performing phylogenetic analysis in microbial genomes (e.g., haploid fungal outbreaks). This service uses the snippy pipeline, which includes FastQC and fastp for quality control and preprocessing of sequencing reads, Snippy for mapping, variant calling, and generating a core SNP matrix (including only SNPs present in all strains in the analysis), IQ-TREE for constructing a phylogenetic tree, and visualization of the phylogenetic tree with iTOL.

    Below are the files that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary>

    <b>Service Notes Description</b>

    When requesting a service in iskylims, researchers must provide essential files and information to ensure accurate variant calling and downstream phylogenetic analysis.

    - **Reference genome**: The genome assembly of the organism being analyzed, in FASTA format. This file will be used for mapping and variant calling.

    - **Sample metadata** (optional): A plain text file containing metadata for the samples (e.g., sample IDs, collection dates, or locations) to facilitate outbreak analysis and improve tree visualization.

    </details>

  - #### **Bacteria: _De novo_ genome assembly and annotation ([Assembly](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/assembly))**

    The Bacterial De Novo Genome Assembly and Annotation Service is designed for assembling and annotating bacterial genomes. This service uses the nf-core/bacass pipeline, which performs FastQC and fastp for quality control and preprocessing of sequencing reads, Unicycler for genome assembly, and QUAST for assessing the quality of the assembled genome. Additionally, kmerfinder is used to detect potential contamination and identify the closest reference genome for improved quality assessment with QUAST. Annotation is performed using prokka and/or bakta.

    Below is the information that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary>

    <b>Service Notes Description</b>

    When requesting a service in iskylims, researchers must specify the bacterial species corresponding to the strain being analyzed. Each service request should correspond to a single bacterial species to streamline and facilitate the analysis process.

   </details>

  - #### **Bacteria:  In-depth analysis of Mycobacterium species genomes (e.g. _M. tuberculosis_. _M. bovis_) ([MTBSeq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/mtbseq))**

    The In-Depth Analysis of Mycobacterium Species Genomes Service is specifically designed for the genomic analysis of Mycobacterium species, such as M. tuberculosis or M. bovis. This service uses the MTBSeq pipeline to perform mapping and variant calling. Additionally, it identifies lineages and variants associated with antibiotic resistance. For outbreak detection, it generates an SNP matrix that can be further analyzed for phylogenetic studies.

    Below are the files and information that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary>

    <b>Service Notes Description</b>

    When requesting this service in iskylims, researchers must provide the following:

    - Metadata file (optional): Provide additional sample information, such as sample IDs, collection dates, or locations, to facilitate the interpretation of lineages and outbreak analyses.

    </details>

  - #### **Bacteria: Plasmid analysis and characterization ([PlasmidID](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/plasmidid))**

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

  - #### **Bacteria: Multi-Locus Sequence Typing (MLST), analysis of virulence factors, antimicrobial resistance, and plasmids characterization ([characterization](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/characterization))**

    The Multi-Locus Sequence Typing (MLST), Analysis of Virulence Factors, Antimicrobial Resistance, and Plasmids Characterization Service provides a comprehensive genomic characterization of bacterial strains. This pipeline utilizes ARIBA for sequence typing (MLST) and antimicrobial resistance (AMR) detection, leveraging the CARD database for identifying resistance genes and mutations. Additionally, AMRFinder Plus is employed for further resistance profiling. Virulence genes are detected using the virulenceFinder and VFDB databases, while plasmid incompatibility typing is performed using plasmidFinder. The pipeline also includes specific typing tools for certain bacterial species, such as emmtyper for Streptococcus pyogenes and TulaTyper for Francisella tularensis.

    Below are the files and information that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary> <b>Service Notes Description</b>
    When requesting this service in iskylims, researchers must provide the following:

    - MLST schema: Specify the MLST schema from PubMLST that should be used for sequence typing. This ensures accurate typing for the bacterial species under analysis.

    </details>

  - #### **Bacteria: Core genome or whole genome Multi-Locus Sequence Typing analysis (cg/wgMLST) ([wgmlst_chewbbaca](https://github.com/BU-ISCIII/buisciii-tools/tree/main/bu_isciii/templates/chewbbaca))**

    The Bacterial Core Genome or Whole Genome Multi-Locus Sequence Typing Analysis (cg/wgMLST) Service is designed for allele calling and phylogenetic analysis based on MLST schemas. This service uses chewBBACA for allele calling and generating the allele matrix, followed by GrapeTree for constructing a Minimum Spanning Tree (MST) to visualize phylogenetic relationships. The analysis can be performed using assemblies provided by the researcher or generated via the De Novo Genome Assembly and Annotation Service described previously.

    Below is the information that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary> <b>Service Notes Description</b>

    When requesting this service in iskylims, researchers must provide:
    - Either an MLST schema from an established database (e.g., PubMLST) or a set of genomes to generate an ad-hoc schema.
    - metadata for the analyzed genomes (e.g., sample IDs, collection dates, or locations) can optionally be provided to enhance the visualization in the Minimum Spanning Tree.

    </details>

  - #### **Viral: Genomic reconstruction, variant calling and _de novo_ assembly ([viralrecon](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/viralrecon))**

    Viralrecon is a bioinformatics analysis pipeline used to perform assembly and intrahost/low-frequency variant calling for viral samples. The pipeline supports both Illumina and Nanopore sequencing data. For Illumina short-reads the pipeline is able to analyse metagenomics data typically obtained from shotgun sequencing (e.g. directly from clinical samples) and enrichment-based library preparation methods (e.g. amplicon-based: ARTIC SARS-CoV-2 enrichment protocol; or probe-capture-based). For Nanopore data the pipeline only supports amplicon-based analysis obtained from primer sets created and maintained by the ARTIC Network. Some examples of viruses analyzed with this pipeline are SARS-CoV-2, mumps virus, monkeypox virus, West Nile virus, etc.

    <details>
    <summary>Required information for service request</summary>
    <br>
    For the correct performance of the pipeline, it is necessary to provide some input documents:

    - **Primers bed file**.
    In case of amplicon-based method, we need to provide a BED file with primer coordinates for the mapping step.

    - **Primers fasta file**.
    Additionally, a fasta file will be necessary if de novo assembly is requested.

    - **[viralrecon_input.xlsx](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/docs/assets/input_datasets/viralrecon/viralrecon_input.xlsx)**

      This document contains 3 different columns:

      - SampleID: Identifier assigned to each sample to be analyzed.
      - Reference: Reference genome (or sequence) to be used to perform the analysis of each sample in the pipeline.
      - Host: Specifies the host organism from which the sequenced sample was obtained.

      Notes:
      - At least one row for every sample must be included in the document.
      - If a sample is required to be analyzed against different references (individually), one row for each one is required.
      - For multifasta documents (e.g. fragmented genomes or custom documents) containing several references, their name should be specified in the Reference column.

    </details>

  - #### **Viral Flu: Influenza fragment reconstruction and variant detection ([IRMA](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/IRMA))**

    The Influenza Fragment Reconstruction and Variant Detection Service focuses on clustering, mapping, and variant calling to generate consensus genomes for viral pathogens, such as Influenza (FLU), RSV, EBOLA, or SARS-CoV-2, using the CDC's IRMA software. This service is designed to handle sequencing data generated from both direct sequencing and amplicon-based approaches, ensuring accurate reconstruction of viral genomes.

    Below are the files and information that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary> <b>Service Notes Description</b>
    When requesting this service in iskylims, researchers must provide the following:

    - **Sequencing approach:** Specify whether the data was generated using a direct sequencing approach or through an amplicon-based method.
    - **Primers file (if amplicons):** In the case of amplicon-based sequencing, provide a multi-FASTA file containing the primer sequences used for generating the amplicons. This is necessary for removing primers during preprocessing to ensure accurate downstream analysis.

    </details>

- ### mRNAseq: Transcriptome sequencing  ([mrnaseq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/rnaseq))

  - #### **Differential Gene Expression (DEG) ([rnaseq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/rnaseq))**

    The RNAseq service performs a quality control (QC), trimming and alignment followed by quantification with [Star](https://github.com/alexdobin/STAR) and [Salmon](https://combine-lab.github.io/salmon/), respectively. After quantification, differntial expression analysis is carried out with [DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html).

    Below are the files that **researchers NEED to provide** when requesting the RNA-seq service.

    <details markdown="1">
    <summary>Required information for service request (genes)</summary>

    **Service Notes Description**

    When requesting a service in iskylims, researchers are required to provide pertinent details, including the type of NGS data intended for analysis. Please be specific when requesting the mRNA-seq service by indicating something like: 'mRNAseq for genes'.

    **[comparatives.txt](./assets/input_datasets/rnaseq/comparatives.txt)**

    The `comparatives.txt`([link to access](./assets/input_datasets/rnaseq/comparatives.txt)) file defines the experimental design for the analysis. It specifies the comparison order, sense, and direction between sample groups. Each comparison requested should have a corresponding line in this file. The file format consists of three columns without headings:

    1. Incremental index representing each comparison.
    2. Treatment group/s.
    3. Control group.

    Example:

      ```Bash
      1 Treatment Control
      2 Treatment       Control
      3 Treatment       Control
      4 Treatment1-Treatment2       Control1-Control2
      ```

    **[clinical_data.txt](./assets/input_datasets/rnaseq/clinical_data.txt)**

    The `clinical_data.txt` ([link to access](./assets/input_datasets/rnaseq/clinical_data.txt)) file is necessary for categorizing the names of samples into comparison groups. This file comprises two columns:

    - **Name:** Sample name.
    - **Group:** Group to which the sample belongs.
    - **Batch** Label that groups samples according to their batch.

    Example:

      ```Bash
         Name    Group  Batch
      ```

    </details>

  - #### **Differential transcript expression (DET) ([rnaseq](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/rnaseq))**
  
    The RNAseq service performs a quality control (QC), trimming and alignment followed by quantification with [Star](https://github.com/alexdobin/STAR) and [Salmon](https://combine-lab.github.io/salmon/), respectively. After quantification, differntial expression analysis is carried out with [fishpond](https://www.bioconductor.org/packages/release/bioc/html/fishpond.html).

    Below are the files that researchers need to provide when requesting the RNA-seq service.

    <details markdown="1">

    <summary>Required information for service request (transcripts)</summary>

      **Service Notes Description**
      When requesting a service in iskylims, researchers are required to provide pertinent details, including the type of NGS data intended for analysis. Please be specific when requesting the mRNA-seq service by indicating something like: 'mRNAseq for transcripts'.

      **[comparatives.txt](./assets/input_datasets/rnaseq/comparatives.txt)**

      The `comparatives.txt` ([link to access](./assets/input_datasets/rnaseq/comparatives.txt)) file defines the experimental design for the analysis. It specifies the comparison order, sense, and direction between sample groups. Each comparison requested should have a corresponding line in this file. The file format consists of three columns without headings:

      1. Incremental index representing each comparison.
      2. Treatment group/s.
      3. Control group.

      Example:

      ```Bash
      1 Treatment Control
      2 Treatment       Control
      3 Treatment       Control
      4 Treatment1-Treatment2       Control1-Control2
      ```

      **[clinical_data.txt](./assets/input_datasets/rnaseq/clinical_data.txt)**

      The `clinical_data.txt` ([link to access](./assets/input_datasets/rnaseq/clinical_data.txt)) file is necessary for categorizing the names of samples into comparison groups. This file comprises two columns:

    - **Name:** Sample name.
    - **Group:** Group to which the sample belongs.
    - **Batch** Label that groups samples according to their batch.

      Example:

      ```Bash
         Name    Group  Batch
      ```

    </details>

  - #### **Differential miRNA expression (DEM) (mirnaseq)**

    The Differential miRNA Expression (DEM) Analysis Service is designed to identify differentially expressed miRNAs from sequencing data. This pipeline uses miRDeep2 for miRNA analysis based on the miRBase database. Differential expression analysis is then performed with edgeR and DESeq to identify miRNAs with significant changes between experimental groups.

    Below are the files and information that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary>

    <b>Service Notes Description</b>

    When requesting this service in iskylims, researchers must provide the following:

    - **Adapter sequence:** Provide the adapter sequence used during library preparation to enable accurate preprocessing of the reads.
    - **Metadata file:** Include a metadata file indicating the experimental groups for differential expression analysis. This should specify the conditions or factors to compare.

    </details>

  - #### **Gene expression changes over a series of time points (timeseries_rnaseq)**
  
    The RNAseq service performs a quality control (QC), trimming and alignment followed by quantification with [Star](https://github.com/alexdobin/STAR) and [Salmon](https://combine-lab.github.io/salmon/), respectively. After quantification, differntial expression analysis is carried out with ad-hoc software/scripts.

- ### Metagenomics and targeted metagenomics

  - #### **Taxonomic based Identification and classification of organisms in complex communities ([taxprofiler](https://github.com/BU-ISCIII/buisciii-tools/tree/main/bu_isciii/templates/taxprofiler))**

    The Taxonomic-Based Identification and Classification of Organisms in Complex Communities Service uses the nf-core/taxprofiler pipeline to perform taxonomic identification of sequenced samples. This service is primarily designed for analyzing metagenomic samples or detecting potential contaminations. The pipeline employs tools such as Mash, Kraken2, Diamond, and Kaiju, generating detailed reports that provide insights into the taxonomic composition of the sample.

    This service does not require any additional input from the user.

  - #### **_De novo_ assembly contigs' alignment to database [BLAST](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/blast_nt)(blast_nt)**

    The De Novo Assembly Contigs' Alignment to Viral Genomes Database Service uses the BLAST tool to align contigs generated from de novo assemblies to a database of viral or bacterial genomes obtained from GenBank or RefSeq. This service is designed to identify or classify viral sequences, providing valuable insights into their origin and characteristics. The analysis can be performed on contigs generated from the De Novo Genome Assembly and Annotation Service (for bacterial genomes) or the Viral Genome Generation Service.

    This service does not require any additional input from the user.

  - #### **Bacteria: 16S rRNA gene analysis to assess bacterial diversity (16s_metagenomics)**

    The 16S rRNA Gene Analysis to Assess Bacterial Diversity Service uses the QIIME2 pipeline to perform metagenetic analysis of 16S rRNA sequencing data. This service is designed to explore bacterial diversity and composition in complex communities. The pipeline is configured based on the specific region of the 16S rRNA targeted during library preparation to ensure accurate analysis and reliable results.

    Below are the files and information that researchers NEED to provide when requesting this service.

    <details markdown="1"> <summary>Required information for service request</summary>

    <b>Service Notes Description</b>
    When requesting this service in iskylims, researchers must provide the following:

    - **Library preparation details:** Specify the targeted region of the 16S rRNA gene (e.g., V3-V4, V4) and any other relevant details about the library preparation protocol to configure the pipeline correctly.
    - **Metadata file (optional):** Researchers can optionally provide a metadata file with information such as sample IDs, experimental groups, collection dates, or locations. This metadata will enhance the analysis of bacterial composition and diversity, allowing for better interpretation of the results.

    </details>

  - #### **Viral:  Detection and characterization of viral genomes within metagenomic data ([pikavirus](https://github.com/BU-ISCIII/buisciii-tools/tree/develop/buisciii/templates/pikavirus))**

    The Detection and Characterization of Viral Genomes within Metagenomic Data Service uses the Pikavirus pipeline to identify and characterize viral genomes present in metagenomic samples. The pipeline employs Mash for rapid identification of viral sequences, followed by confirmation using mapping-based statistics. Viral identification is performed using the viral genome database from GenBank (NCBI), generating a comprehensive report with detailed results.

    This service does not require any additional input from the user.

- ### Bioinformatics consulting and training

  Once requested any of the services in this category, a meeting with the bioinformatics team will be organized to discuss needs and plan the service approach.  

  - #### **Bioinformatics analysis consulting**

    Service providing tailored support for designing, executing, and interpreting bioinformatics analyses. Once requested, a meeting with the bioinformatics team will be organized to discuss needs and plan the analysis approach.

  - #### **In-house and outer course organization**

    Support for organizing bioinformatics courses and workshops, either hosted internally or externally, tailored to the specific needs of participants.

  - #### **Student training in colaboration: Master thesis, research visit,...**

    Collaboration opportunities for students to gain bioinformatics expertise through master’s thesis projects, research visits, or similar training programs in coordination with the bioinformatics team.

- ### User support

  - #### **Installation and support of bioinformatic software on Linux OS**

    Assistance with installing and configuring bioinformatics software on Linux-based systems, ensuring functionality and compatibility with research workflows.

  - #### **Installation and access to Virtual machines in the Unit server containing bioinformatic software**

    Support for accessing and setting up virtual machines hosted on the Unit's server.

  - #### **Code snippets development**

    Development of custom scripts or code snippets to automate or streamline bioinformatics tasks, tailored to user requirements.
