# How to perform the mRNAseq Service

This is a brief tutorial on how to perform an mRNAseq Service as a member of the BU-ISCIII!.

After creating the service folder in `services_and_colaborations` using the [buisciii-tools](https://github.com/BU-ISCIII/buisciii-tools), your **service folder** should look like this:

```bash
cd /data/ucct/bi/services_and_colaborations/*/SRVXXXYYYY_YYYYMMDD_RNASEQZZZ_researcher_S/

tree -L 1 .
.
├── ANALYSIS
├── DOC
├── RAW
├── REFERENCES
├── RESULTS
└── TMP
```

If so, you are ready to proceed.

## Step-by-step guide.

### RNA-seq analysis

1. Go to `ANALYSIS/` in the service folder:
  - Execute the `lablog_rnaseq`.
    ```bash
    cd /data/ucct/bi/services_and_colaborations/virology/SRVXXXYYYY_YYYYMMDD_RNASEQZZZ_researcher_S/ANALYSIS
    bash lablog
    ```

  - Copy files in the folder service from `services_and_colaborations` to `scratch_tmp/`.
    ```bash
    buisciii --log-file SRVXXXYYYY.1.tool.log scratch --direction service_to_scratch SRVXXXYYYY.1
    ```
    The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service.

2. Go to `ANALYSIS/${DATE}_ANALYSIS01_RNASEQ` in the service folder copied to `scratch_tmp/` and execute the `lablog`.
  ```bash
  cd /data/ucct/bi/scratch_tmp/bi/SRVXXXYYYY_YYYYMMDD_RNASEQZZZ_researcher_S/ANALYSIS/${DATE}_ANALYSIS01_RNASEQ
  bash lablog
  ```

3. Run the analysis:
  - Load dependencies:
    ```bash
    module load Nextflow singularity
    ```

  - Deploy the mRNAseq analysis with slurm
    ```bash
    bash _01_run_rnaseq.sh
    ```

After the process is completed, a new folder named `01-${DATE}_rnaseq` is generated. This folder contains the results of the mRNAseq analysis. Among the most relevant files to check and review are:

- `multiqc/star_salmon/multiqc_report.html`: An HTML report displaying quality control metrics for raw reads alignment, transcript expression quantification, strand-specificity checks, and more. Here are the **key sections** in the report to focus on:

  - **FASTQC**: Assess the quality of reads in the FastQC section, which presents the mean quality value across each base position in the read.

  - **STAR**: Review the percentage of uniquely mapped reads mapping.
  
  - **`salmon.merged.gene_counts.tsv`**: This file contains the merged counts data.

  - **RSeQC**: Check the predominant **strandedness** value within the Infer experiment section and ensure it matches the information in the samplesheet (`SRVXXXYYYY_YYYYMMDD_RNASEQZZZ_researcher_S/ANALYSIS/${DATE}_ANALYSIS01_RNASEQ/samplesheet.csv`).

>[!WARNING]
>**If the strandedness value does not match, then update the value in the `samplesheet.csv` file and re-deploy the mRNAseq analysis.**


### Differential Expression Analysis

4. Go to `02-differential_expression/`:
  - Explore the files in this directory. Among them, the `comparatives.txt` file contains the experimental design, which will be used to perform differential expression analyses (each line corresponds to an experimental analysis). The `clinical_data.txt` is a two-column dataframe used to assign samples to groups in the experiments.
    ```bash
    .
    ├── clinical_data.txt
    ├── comparatives.txt
    ├── differential_expression.R
    └── lablog
    ```
  - Load the dependencies: 
    ```bash
    module load R/4.1.3
    ```

  - Execute the `lablog`.
    ```bash
    bash lablog
    ```

  - A folder is generated for each differential expression analysis defined in `comparatives.txt`:
    ```bash
    .
    ├── 1_Treatment_Control
    ├── 2_Treatment
    ├── 3_Treatment
    ├── 4_Treatment1-Treatment2
    ├── clinical_data.txt
    ├── comparatives.txt
    ├── Control
    ├── Control1-Control2
    ├── differential_expression.R
    └── lablog
    ```

    > Note: The number of folders containing differential expression analysis varies according to what is specified in the experimental design. See the `comparatives.txt` file. The number of folders will be equal to the number of differential gene expression analyses requested.

5. Run differential expression analysis.
  - To run the differential expression analysis, go to the experiment folder (in this example it will be  1_Treatment_Control)
    ```bash
    cd 1_Treatment_Control/
    ```
  - Execute the `lablog`. This will generate all necesary files to perform the analysis.
    ```
    bash lablog
    ```
  - Deploy the differential expression analysis with slurm by running:
    ```bash
    bash _01_deseq2.sh
    ```

Once the process is finished, within the differential expression directory (remember, in this example we are using `1_Treatment_Control/`), you will see the results of the differential expression analysis for the specific compairson. Among the files, you should pay speccial attention to:

- `logs/DESEQ2.${DATE}.log`: Here, you can find information on the **number** of differentially expressed genes and their regulation status (up/down).

- `Differential_expression/DESeq2/Differential_expression.csv`: This file contains a dataframe with the results of the differentially expressed genes, including parameters such as log2fold change, p-value, and padj.

- `Differential_expression/DESeq2/heatmapCount_top20_differentially_expressed.pdf`: This PDF file contains a heatmap image displaying the top 20 most expressed genes clustered by sample distance.

- `99-stats/Quality_plots/DESeq2/plotPCA.pdf`: This PCA image illustrates the distribution of samples according to their distance. Ideally, samples from the same group should cluster together.


### mRNAseq REPORT TEMPLATE (TEAM STANDUP)

Once you have finished the analysis and analyzed the results **you need to report to the rest of the members**. Use this **template** to do it:

**SRVXXXYYYY_YYYYMMDD_RNASEQZZZ_researcher_S**
* **Servicio**: [service name]
* **Carrera**: [WGS run name / WGS project name]
* **Strandness**: Sense/Antisense/No strand-specific
* **Muestras**: [number of samples analyzed in the service]
* **Solicitante**: [name of the person who requested the service]
* **Laboratorio**: [laboratory alias of the requester]
* **Estado**: [status of the service]
* **Cantidad de lecturas**: [the median amount of reads]
* **Calidad de las lecturas**: [quality of reads]
* **Resultados generales**:
  * Porcentaje de mapping: [mapping percentage in the multiqc report]
  * RSeQC infer experiment: [is the strandedness what should be expected?]
* **Análisis de expresión diferencial**:
  * Number of differentially expressed genes/transcripts.
* **Análisis de datos exploratorio**:
  * Relevant notes regarding sample groups and outliers in the PCA plot.

