# How to perform a MAG Service

This is a brief tutorial on how to perform a MAG Service as a member of the ISCIII's Bioinformatics Unit! MAG is a associated with _Taxonomic based Identification and classification of organisms in complex communities_ service from service catalog, but usually, MAG is part of other services like [IRMA](lint/to/flu/service), [viralrecon](https://github.com/BU-ISCIII/BU-ISCIII/wiki/SARS-CoV-2-service), or [PikaVirus](/link/to/pikavirus/).

[nf-core/mag](https://github.com/nf-core/mag) is a bioinformatics best-practise analysis pipeline for assembly, binning and annotation of metagenomes. This service aims to perform taxonomic classification of reads using k-mer method with Kraken2 to identify which viruses/bacteria (and fungi/plants if the PlusPF/PlusPFP database is used) are present. This will allow us to identify contaminations if the species in the sample are known, or to get an idea of the species present in a metagenomic sample. It is worth noting that this usage of the nf-core/mag pipeline does not allow for strain/subtype identification, so it only provides a general idea of the sample composition.

Let's get started with the service. When performing a MAG service, it's likely that the service folder, resolution, and acronym are already created as part of these other services. Typical acronyms for a service that includes MAG analysis are:

- GENOME** (for example: GENOMEFLU, GENOMERSV, GENOMEEV...)
- VIRAL-DISCOVERY
- SARSCOV2

In  the case that only the service _Taxonomic based Identification and classification of organisms in complex communities_ was requested in iSkyLIMS, the most appropriate acronym is VIRAL-DISCOVERY. To perform this manual, we will assume that the service only contains MAG.

First of all, follow the [first steps to create a service](/link/to/tools/and/iskylims/TODO). Once the `new-service` is finished, you'll have a new folder with 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP` (as explained [here](https://github.com/BU-ISCIII/BU-ISCIII/wiki/bioinformatics#33-services_and_collaborations)). We should check the following folders before going any further:

- `DOC`: This folder should contains a `mag.config` file with the specific configuration for nf-core/mag in our HPC.
- `RAW`: Check that the number of files contained within the RAW folder is equal to the number of samples specified in [iskyLIMS](https://iskylims.isciii.es/) x 2, in the case that they are paired-end reads.

If everything is OK, we can get into the `ANALYSIS` folder and we'll find the following items inside:

- `lablog_mag`: an executable file that creates the 00-reads folder, moves inside, creates symbolic links to the reads renaming them and renames the `ANALYSIS0X_MET` folder.
- `samples_id.txt`: a `.txt` file containing all the sample names, one per line, so there will be as many lines as samples associated with our service.
- `ANALYSIS0X_MAG`: Folder with the main MAG analysis files.

First of all, let's **check in the `lablog_mag` if the renaming of the `ANALYSIS0X_MET` is correct**:

- If MAG analysis is the only analysis in the service, the folder will be `DATE_ANALYSIS01_MET`
- If MAG analysis is part of other services, you will have to sum as many numbers as other analysis you have prior to MAG. Usually it is `DATE_ANALYSIS02_MET`

Now we can execute the lablog:

```bash
bash lablog_mag
```

> [!WARNING]
> If MAG is not the only analysis in your service, don't forget to run the other `lablogs` before the next steps.

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch` as explained [here](/link/to/tools/and/iskylims/TODO)

Once this function is finished, we should go into the `scratch_tmp` folder and the specific `ANALYSIS` folder associated with our service. Once we're inside, we will see the following folders/files:

- `lablog`: will create symbolic links to `00-reads` and the `samples_id.txt` file, and creates `mag.sbatch` and `_01_run_mag.sh` files.
- `99-stats`: We need this folder because default MAG's MultiQC does not take Kraken2's output into the final report, and we need it.

Let's execute the `lablog` file:

```bash
bash lablog
```

Now, we should check we've loaded all the needed dependencies and perform the metagenomic analysis:

```bash
module load Nextflow singularity
bash _01_run_mag.sh
```

After this, the analysis will start, and we'll be able to check the status of the process with:

```bash
tail -f DATE_mag.log
```

Once checked everything has finished OK in the `DATE_mag.log`, it's time to execute the content of the `99-stats` folder:

```bash
cd 99-stats
bash lablog
module load MultiQC
bash _01_run_multiqc.sh
```

Once this MultiQC process has finished, we can execute this other script to clean the symbolik links in the folder:

```bash
bash _02_unlink.sh
```

Once all the process is finished, within the DATE_ANALYSIS0X_MAG folder, we'll have the following content:

- `DATE_mag/`: Results of the nf-core/mag pipeline.
  - `multiqc/multiqc_report.html`: MultiQC's HTML report of the MAG pipeline.
  - `QC_shortreads`: Quality control results (fastQC, fastp,...)
  - `Taxonomy/kraken2/<sample_name>/taxonomy.krona.html`: Kraken2 results plotted in circos using Krona.
- `99-stats/multiqc_report.html`: HTML report with the top5 species found in kraken2 among all samples.

In this service we should check:

- `multiqc/multiqc_report.html`: FastQC/fastp reports to assess the good/bad quality of the reads, the presence of adaptaers, etc...
- `99-stats/multiqc_report.html`: To ensure that there are no contaminations or unespected species.
  - If any sample has a lot of `other` species in this MultiQC report, don't hesitate to check its `Taxonomy/kraken2/<sample_name>/taxonomy.krona.html` because it may have specific species differing from other samples.
  - If a sample has a lot of `unclasified` species, check that `Taxonomy/kraken2/<sample_name>/taxonomy.krona.html` contains a lot of `No hits` in the report. This might be explained by two different reasons:
    1. The organisms present in the sample are not present in the database used for nf-core/mag pipeline.
    2. The is a lot of the so called `Space junk` (a.k.a. basura espacial), which means that the sequencing process was not so good...

Once all the service results are revised in the `scratch_tmp` folder, you can continue with the next BU-ISCIII tool: `finish` as explained [here](/link/to/tools/and/iskylims/TODO).

When everything is propperly coppied into the SFTP folder, you can put the update in the `update_servicios` channel's _Team Standup Bot_, with the following template:

- _**SRVXXXXXX - ACRONYMXX**_ :no_entry: = No empezado, parado o esperando   :running: = Corriendo   :mag: = En revisión   :clipboard: = Finalizado copiandose a SFTP   :white_check_mark: = Finalizado y copiado a sftp
  - **Servicio**: Taxonomic based Identification and classification of organisms in complex communities (MAG)
  - **Número de muestras**: # Numero de muestras
  - **Estado**: No empezado/Corriendo/En revisión/Finalizado copiandose a SFTP/Finalizado y copiado a sftp
  - **Instrumento y longitud**: Ej.: NovaSeq (2x150)
  - **Cantidad de lecturas**: Ej.: 0.8M - 91.2M
  - **Calidad general**: Buena/Mala/Indicar incidencias específicas
  - **Resultados generales**: Breve resumen de los resultados obtenidos a grandes rasgos, describiendo los resultados más relevantes del `99-stats/multiqc_report.html`.
  - **Resultados por muestra**: _Indicar solo los resultados ANOMALOS_. Algunos ejemplos:
    - La muestra XXX contiene X virus (si es que no se espera ese virus en esa muestra)
    - La muestra XXX se ha secuenciado con mala calidad y ha perdido muchas lecturas en el preprocesamiento.
    - La muestra XXX no tiene lecturas suficientes para realizar el análisis.

When the revisors give the OK to the service results, you can finish the service with the next BU-ISCIII tool: `delivery` as explained [here](/link/to/tools/and/iskylims/TODO).
