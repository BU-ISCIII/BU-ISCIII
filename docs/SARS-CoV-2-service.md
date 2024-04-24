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

When there's a mix of full-numerical and strings in sample IDs (e.g. `87439.fastq.gz` and `SARS_01.fastq.gz`) the pipeline may crush in `MULTIQC` step. This is caused because there's a bug with MultiQC ([MultiQC issue](https://github.com/nf-core/viralrecon/issues/345)) that can be temporarily fixed by adding any non-numerical character to the sample IDs. Nevertheless, you can follow the instructions in [this tutorial](https://drive.google.com/drive/u/0/folders/1-GafpZR2HVlecNaAsXIslK3aecHplD4z) to properly correct this error.