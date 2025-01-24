# PlasmidID service

**PlasmidID** is a **mapping-based**, **assembly-assisted** **plasmid** **identification** tool that analyses and provides a graphic solution for **plasmid identification**.

**PlasmidID** is a computational pipeline implemented in BASH that:
1. Maps Illumina reads over several plasmid database sequences.
2. The k-mer-filtered, most-covered sequences are then clustered by identity to avoid redundancy, and the longest ones are used as scaffolds for plasmid reconstruction.
3. Reads are assembled and annotated by automatic and specific annotation procedures.
4. All the information generated from mapping, assembly, annotation and local alignment analyses is gathered and accurately represented in a circular image which allows the user to determine the plasmidic composition present in any bacterial sample.

This software can be accesed via [**GitHub**](https://github.com/BU-ISCIII/plasmidID).

Through this guide, **you'll learn how to use PlasmidID for plasmid identification**, as part of an outbreak service, since this is usually the context in which this service is requested.

## Introduction

### Service request

Within which context is this service carried out? In general, the researcher that requests this service provides an external file (usually an Excel file) indicating the samples that must be analysed, which species they correspond to, etc., unless there aren't a lot of samples or they are all related to the same species.

Considering this information, they will generally ask for the **assembly** of the samples, followed by the **identification** of **virulence factors**, **antibiotic resistances** and **plasmids**, and an **SNP analysis**:
* For the **assembly** of the samples, the bioinformatics procedure that must be carried out is explained in detail the **[Assembly service](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) guide**.
* For the **identification of virulence factors, resistances and plasmids**, please follow the instructions from the [**Characterization guide**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* For the identification and characterization of **plasmids**, this procedure is done bioinformatically by means of **PlasmidID**. Its usage will be explained **in this guide**.
* For the **SNP analysis** of the samples, this is done bioinformatically with the **[SNIPPY](https://github.com/tseemann/snippy)** software, please follow the instructions from the [**Snippy guide**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service).

## Bioinformatics procedure

>[!WARNING]
>As stated before, the PlasmidID service is usually not done alone, but along with de novo assembly, characterization and snippy. Therefore, you'll most likely already have created the service folder, which will still have the acronym associated to the Characterization service, as indicated below.

> **Ignore the following if the service folder has already been created** (which is the most probable thing):<br><br>
>Taking the previous information into account, the first thing we need to do is take the service and click on _**Add resolution**_ in **[iSkyLIMS](https://iskylims.isciii.es/)** after logging in with your user and password. For this to happen, you need to specify the estimated delivery date, your user and a service acronym.
>
>In order to know which acronym to use for this new resolution, log into your WS user and execute the following commands:
>
>```shell
>cd /data/ucct/bi/services_and_colaborations/CNM/bacteriology
>ll -tr
>```
>
>Once these commands have been executed, you'll get a list of all the folders (each one related to one service) contained within the `services_and_colaborations` folder, sorted by time in a reverse order. Therefore, you'll see in the last place the last service that was delivered.
>
>At this point, you'll have to consider what kind of service has been requested in this case. In general, follow these guidelines:
>* If all samples correspond to a specific bacterial species, within the context of an outbreak, the service acronym should have the following structure: `SPECIESOUTBREAKXXX`. For example, if the outbreak corresponds to *Bacillus cereus* and this is the first outbreak we analyse in relation to this species, the service should be called `BCEREUSOUTBREAK001`.
>* If the service is related to one or several species, but it is not within the context of an outbreak, the service should be called after the following structure: `WGSORGANISMXX`. For example, if we have samples from several species from the same genus (like *Burkholderia*), we should call the service `WGSBURKHOLDERIA01` (if it's the first service related to *Burkholderia* species).
>* If the service corresponds to samples from several species, not necessarily from the same genus, and therefore not all species can be categorised inside one name, we should call the service after the following structure: `CHARACTERIZATIONXX`. For example, if our service includes samples with a wide variety of species from different genera, we should call the service `CHARACTERIZATION01` (if it's the first service related to multiple species).
>
>Regardless of the acronym that has been assigned to the service, your service acronym will be the last service's number + 1. For example, if you have chosen the `CHARACTERIZATION` acronym for your service and the last folder after executing `ll -tr` is `SRVCNM1200_20240823_CHARACTERIZATION07_svaldezate_S`, the service acronym for your new service will be _**CHARACTERIZATION08**_. Specify this in the form that appears after clicking on _Add resolution_, and click on **_Accept_**.

Now, considering you've already created a buisciii-tools micromamba environment and installed the tools (this will be necessary eventually) in your local PC, log into your HPC user.

Once you're logged in, go into the `services_and_colaborations` folder:

```shell
cd /data/ucct/bi/services_and_colaborations/CNM/bacteriology/
ll -tr
```

Now, let's execute the first BU-ISCIII tool: `new-service`, where you'll need to specify the resolution ID associated to this service.

```shell
buisciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X
```

The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results).  

Once `new-service` is executed, you'll be asked:

* `Do you want to skip folder creation?`:
  * **If the service folder has not been created yet in the `services_and_collaborations` folder**, answer **NO**.
  * **If this is not the first resolution associated with the service or another service has already been performed for the current resolution**, answer **YES**.

* Next, specify `plasmidid`, since this is the service we're running.

Once the `new-service` tool is finished, you'll have a new folder in `services_and_colaborations` with the following structure: `SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S`. Your service will now appear within the _**In progress**_ tab in [iSkyLIMS](https://iskylims.isciii.es/).

If we get into this folder, we'll find 6 folders: `ANALYSIS`, `DOC`, `RAW`, `REFERENCES`, `RESULTS` and `TMP`. We should check, before going any further, that the number of files contained within the `RAW` folder is equal to the number of samples specified in [iSkyLIMS](https://iskylims.isciii.es/) x 2, since these are paired-end reads.

If everything is OK, we can then get into the `ANALYSIS` folder and we'll find, apart from folders and files from other services done previously, the following `lablog` file inside:

* `lablog_plasmidid`: an executable file that renames the `ANALYSIS02_PLASMIDID` folder so that it contains the analysis date. Please remember to change the number associated to the word `ANALYSIS` accordingly, depending on whether other services have been done previously or not.

Let's execute the `lablog_plasmidid` file:

```shell
bash lablog_plasmidid
```

Once this file has been executed, please take into consideration that this service is usually performed along with other pipelines (normally **assembly**, **characterization** and **snippy**), so **run all the necessary `lablog` files before moving on to the next BU-ISCIII module**.

After executing this file, if everything is OK, we can now proceed with the next BU-ISCIII tool: `scratch`. This tool will copy the content from `services_and_colaborations` to the `scratch_tmp` folder contained within `/data/ucct/bi`, since this `scratch_tmp` folder will be the one used for the analysis. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

```shell
buisciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
```

Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).

Once `scratch` is executed, you'll be asked:

* `Direction of the service`: in this case, we want to copy our files from service to scratch, so we have to select the `service_to_scratch` option.

Once this function is finished, we should go into the `scratch_tmp` folder and the specific folder associated with our service:

```shell
cd /data/ucct/bi/scratch_tmp/bi/SRVCNMXXX_YYYYMMDD_CHARACTERIZATIONXXX_researcher_S/ANALYSIS/DATE_ANALYSIS03_PLASMIDID
```

> [!WARNING]
> Please note that the PlasmidID service is usually performed along with the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service), the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) and the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service); that's why this folder is called `ANALYSIS03`.
> <br><br>During the assembly service, **we should have saved the trimmed sequences**, since **they will be needed** during the PlasmidID procedure.
><br><br>Please remember to **unzip** the **assemblies** obtained after running **unicycler**, since these assemblies will be used by PlasmidID so that assembly of the reads is not done again unnecessarily, by running `gzip -dk *.fastq.gz`.

Once we're at this point, and before executing anything else, we should load all the necessary dependencies:

```shell
module load singularity
```

If everything is OK, we can then get into the `ANALYSIS` folder and we'll find the following items inside:
* `lablog`: this file will create a new folder called logs, a symbolic link to `samples_id.txt` and two scripts that will be run sequentially:
  * `_01_plasmidID.sh`: this script will run plasmidID for the samples indicated on the `samples_id.txt` file, using the processed reads as input. A folder called NO_GROUP will be created, which will store several subfolders (each one for each sample) with all the intermediary files as well as an .html and .tab report for each sample.
  * `_02_summary_table.sh`: this script will collect the results from all samples and merge all the information in one report file, both in .html and .tab formats.
* `plasmidID_annotation_config_file.txt`: this file is used by PlasmiID to perform the annotation.

Run both scripts in order and check the logs created in the log files for any error before running the next script, and debug any problem you could find.

Once you have run all the required scripts, you should go to the `RESULTS` folder and have a file called `lablog_plasmidid_results` (usually along with `lablog_assembly_results`, `lablog_snippy_results` and `lablog_characterization_results`).

 The `lablog` file corresponding to the PlasmidID service does the following after being run:
  * Creates a folder ending in *entrega01*.
  * Creates, inside this folder, a subfolder called `plasmidid`.
  * Inside this subfolder, it creates symbolic links to all `.png` and `.html` files from all samples.

### Results validation

- If you are unfamiliar with PlasmidID results, please check its output documentation in BU-ISCIII tools template **reports** and its [**results interpretation**](https://github.com/BU-ISCIII/plasmidID/wiki/Understanding-the-image%3A-track-by-track) **manual**.
- In order to check the succesful completion of the pipeline you need to check:
  - The PlasmidID .log file for each sample; here you can see if there is any sample for which no plasmids were found. This should be reported when delivering the service.
  - `NO_GROUP/SAMPLE_NAME/images`: check this folder and open some images so you can see if some plasmids have been correctly found. You can check [**this manual**](https://github.com/BU-ISCIII/plasmidID/wiki/How-to-chose-the-right-plasmids) for this task.
  - Finally, if you have more than one sample, you should check the summary results file created in the root of `NO_GROUP` folder.

## What should I do after I've run all the necessary scripts?

Once we are done with the service (including the assembly, characterization and snippy procedures, since this is the usual case), we'll have to review the results from each procedure and add all the relevant information into an Excel file. For this, we use a **template** that you can find [**here**](https://docs.google.com/spreadsheets/d/1m_hnCGNgtWcoJAjs_91BkmfJn6CNr4yO/edit?usp=drive_link&ouid=108428245306738036878&rtpof=true&sd=true).

In this Excel file, we can find the following sheets:
* **summary**: this sheet contains information regarding the assembly performed previously (check the [**Assembly service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Assembly-service) page), Kmerfinder, the MLST profile identified for the samples and the mapping procedure done against the reference used for snippy.
  * For the **MLST profile**, please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) page.
  * For the **Kmerfinder** columns `07-kmerfinder_best_hit_# Assembly`, `07-kmerfinder_best_hit_Accession Number` and `07-kmerfinder_best_hit_Description`, please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) page.
  * The **MAPPING** columns are relative to the snippy service. Please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service) page.
  * The **ASSEMBLY** columns are relative to the assembly service. Please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service) page.
* **snpmatrix_all_pos**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service).
* **snpmatrix_all_pos_pairs**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service).
* **snpmatrix_core**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service).
* **snpmatrix_core_pairs**: please check the [**Snippy service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Snippy-service).
* **plasmids**: this sheet is filled in combining the results from **ARIBA with plasmidfinder** and from **PlasmidID**. For each sample, there is a .tab report that you should open. You'll find 8 columns: `id`, `length`, `species`, `description`, `fraction_covered`, `contig_name`, `percentage` and `images`. Copy all columns except for the `images` column and paste them into the `plasmids` sheet in the Excel summary file.
* **virulence**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* **Resistance result**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* **AMRFinderPlus Resistance result**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).
* **MLVA**: please check the [**Characterization service**](https://github.com/BU-ISCIII/BU-ISCIII/wiki/Characterization-service).

Fill in the Excel template following the previous instructions, name it according to the structure `summary_outbreak_species.xlsx`, and then do the following:
* Being logged in your Google account, go to the **buisciii_shared** Drive folder.
* Go to **ALERTS**.
* Create a new subfolder called after the organism of interest followed by the date in which the service was requested. For example, if the service that we are carrying out is relative to *Brucella melitensis* and the service was requested on 28/10/2024, the new folder will have this name: `BMELITENSIS_20241028`.
* Go inside this subfolder and store the Excel report in here.

Once the Excel file has been completed with all the necessary information and has been updated into the corresponding Drive folder, **save a copy of this Excel file** both inside the `RESULTS/DATE_entrega01` folder and the `ANALYSIS` folder of the service. 

You can then proceed with the copy of the service to `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` and `/data/ucct/bi/sftp`.

---

If everything is correct and all the necessary files and links have indeed been generated, you can proceed with the service completion.

To do this, execute the **finish** module of buisciii-tools. Please make sure the .log file is saved within the **`DOC`** folder of the service. If this is not the case, please move this file into this folder manually.

    $ buisciii --log-file SRVCNMXXX.X.finish.log finish SRVCNMXXX.X

This module will do several things. First, it cleans up the service folder, removing all the folders and files than are not longer needed and take up a considerable amount of storage space (in **PlasmidID**, this folder is `01-preprocessing/trimmed_sequences`. Besides, all `mapping/sample_name.sorted.bam` and `kmer/database.msh` files should be deleted as well). Then, it copies all the service files back to its `/data/ucct/bi/services_and_colaborations/CNM/bacteriology/` folder, and also copies the content of this service to the researcher's sftp repository.

In order to complete the delivery of results to the researcher, you need to run the **bioinfo-doc** module of the buisciii-tools. To do so, you have to unlogin your HPC user and run it directly from your WS, where you have mounted the `/data/ucct/bioinfo_doc/` folder.

    $ buisciii --log-file SRVCNMXXX.X.tool.log bioinfo-doc SRVCNMXXX.X

Remember to save the logs with the corresponding name (i.e. `SRVCNMXXX.X.service_info.log` or `SRVCNMXXX.X.delivery.log`).

This module will be executed twice. The first time, select the **service_info** option, and the next time select the **delivery** option. There is the option to add delivery notes (by prompt or by providing a file) during its execution.

>[!WARNING]
> When performing an **outbreak** service (as mentioned before, this PlasmidID service is usually not done alone, but along with the assembly, snippy and characterization procedures), the delivery message is in general too long to be included as a .txt file during the delivery procedure. Therefore, for this kind of services, reply the following when these questions are asked on the terminal:
>1. Do you wish to provide a text file for delivery notes?: type n.
>2. Write some delivery notes: leave it blank, by pressing Enter.
>3. Do you want to add some delivery notes to the e-mail?: type n.
>4. Do you want to send e-mail automatically?: type n.
>
>The service_info and delivery .pdf files will have been created but the e-mail won't have been sent. This will be done manually by sending the service report that you can see in the last section of this manual.

Lastly, once the service has been delivered and the e-mail has been sent, remember to remove all the files related to this service from `scratch_tmp`:

    $ buisciii --log-file SRVCNMXXX.X.tool.log scratch SRVCNMXXX.X
    $ remove_scratch

## Outbreak report template

This is just a general template, but feel free to do all the adjustments you need.

```
En este servicio se ha realizado ensamblado de novo, caracterización de resistencias, análisis de factores de virulencia, análisis de plásmidos, análisis de SNPs y análisis MLST.

Adjuntamos tabla resumen (summary_outbreak_XXXX.xlsx) de los resultados de los análisis actualizados, hemos realizado:
- Summary:
    - Análisis MLST.
    - Análisis de identificación de especie y contaminaciones con Kmerfinder.
    - Estadísticas de mapeo de las muestras frente a la referencia XXXX.
    - Ensamblado y anotación de novo.
- Análisis filogenético con matriz de SNPs. Aquí hay dos resultados:
   - Sin filtrar zonas recombinantes (snpmatrix_all_pos y snpmatrix_all_pos_pairs).
   - Filtrando zonas recombinantes (snpmatrix_core y snpmatrix_core_pairs).
- Análisis de plásmidos.
- Análisis de factores de virulencia.
- Búsqueda de genes de resistencia con las bases de datos CARD y AMRFinderPlus.
- Análisis MLVA.

Además, en la carpeta RESULTS, encontrarás todos los resultados que están presentes en la tabla: 
├── assembly
│   ├── assemblies: ficheros fasta de los ensamblados.
│   ├── kmerfinder_summary.csv: resultados de kmerfinder para las muestras.
│   ├── multiqc_report.html: report en html de la calidad del ensamblado y la anotación para las muestras.
│   ├── quast_XXXX_report.html: resultado de quast con la calidad del ensamblado frente a dicha referencia.
│   ├── quast_global_report.html: resultado global de quast con la calidad del ensamblado.
│   └── summary_assembly_metrics_mqc.csv: tabla resumen excel con resultados de la calidad del ensamblado.
├── characterization
│   ├── amrfinderplus: resultados de resistencia a antibióticos con AMRFinderPlus.
│   ├── ariba_card.csv: resultados de resistencia a antibióticos con ariba y base de datos CARD.
│   ├── ariba_plasmidfinder.csv: resultados de identificación de plásmidos y base de datos PlasmidFinder.
│   ├── ariba_vfdb_full.csv: resultados de genes de virulencia con ariba y base de datos VFDB.
│   ├── ariba_mlst_full.tsv: resultado MLST.
├── snp
|   ├── mapping_stats_summary.txt: estadísticas de mapping.
|   ├── phylo.iqtree.bootstrap.nwk: árbol en formato newick, puede ser usado para ver el árbol en MEGA, Figtree, iTOL...
|   ├── clean.core.iqtree.bootstrap.nwk: árbol en formato newick, habiendo filtrado zonas recombinantes, puede ser usado para ver el árbol en MEGA, Figtree, iTOL...
|   ├── variants_stats.txt: estadísticas de número de variantes.
|   |── wgs_metrics_all_filtered.txt: estadísticas más extensas de mapping.
├── plasmidid:
|   ├── <sample_name>_<plasmid_id>.png: Figura de reconstrucción del plásmido para una muestra dada. Se incluye anotación de resistencias.
│   ├── <sample_name>_summary.png: Imagen con el gráfico resumen de todos los plásmidos identificados para una muestra dada.
|   ├── <sample_name>_final_results.html: Fichero HTML con los resultados de PlasmidID para una muestra dada.
├── mlva:
│   ├── MLVA_analysis_assemblies.csv: tabla que contiene los valores MLVA para cada ensamblado en todos los loci del análisis.
│   ├── assemblies_mismatchs.txt: diferencias para cada locus (solo loci con diferencias).
│   ├── assemblies_output.csv: tabla que contiene toda la información del análisis, posición de los primers en cada match, tamaño del insert, número de diferencias (mismatches), etc.
│   ├── predicted-pcr-size-table-assemblies.csv: tabla que contiene los tamaños de PCR predichos.
└── summary_outbreak_XXXX.xlsx: Tabla resumen adjunta en el correo.
```
---

## Troubleshooting

### I got an error message saying `KeyError: 'images'` when running `_02_summary_table.sh`, what should I do?

When running `_02_summary_table.sh`, you might get an error like the following:

```
CREATING SUMMARY REPORT (Thu Jul  4 21:16:04 CEST 2024)
 An html report with miniatures of the images will be generate with useful statistics to determine the correct plasmids in the sample.
Namespace(group=False, input_folder='/scratch/bi/SRVCNM1155_20240614_WGSPANAEROBICUS01_svaldezate_S/ANALYSIS/20240704_ANALYSIS11_PLASMIDID/NO_GROUP/20240349')
Creating summary
Wrong number of items passed 7, placement implies 1
Traceback (most recent call last):
  File "/usr/local/lib/python3.7/site-packages/pandas/core/indexes/base.py", line 3080, in get_loc
    return self._engine.get_loc(casted_key)
  File "pandas/_libs/index.pyx", line 70, in pandas._libs.index.IndexEngine.get_loc
  File "pandas/_libs/index.pyx", line 101, in pandas._libs.index.IndexEngine.get_loc
  File "pandas/_libs/hashtable_class_helper.pxi", line 4554, in pandas._libs.hashtable.PyObjectHashTable.get_item
  File "pandas/_libs/hashtable_class_helper.pxi", line 4562, in pandas._libs.hashtable.PyObjectHashTable.get_item
KeyError: 'images'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/usr/local/lib/python3.7/site-packages/pandas/core/generic.py", line 3826, in _set_item
    loc = self._info_axis.get_loc(key)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/indexes/base.py", line 3082, in get_loc
    raise KeyError(key) from err
KeyError: 'images'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/local/bin/summary_report_pid.py", line 465, in <module>
    main()
  File "/usr/local/bin/summary_report_pid.py", line 458, in main
    final_individual_dataframe = include_images(input_folder, summary_df)
  File "/usr/local/bin/summary_report_pid.py", line 135, in include_images
    summary_df['images'] = summary_df.apply(lambda x: image_finder(x, sample_folder), axis=1)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/frame.py", line 3163, in __setitem__
    self._set_item(key, value)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/frame.py", line 3243, in _set_item
    NDFrame._set_item(self, key, value)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/generic.py", line 3829, in _set_item
    self._mgr.insert(len(self._info_axis), key, value)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/internals/managers.py", line 1203, in insert
    block = make_block(values=value, ndim=self.ndim, placement=slice(loc, loc + 1))
  File "/usr/local/lib/python3.7/site-packages/pandas/core/internals/blocks.py", line 2742, in make_block
    return klass(values, ndim=ndim, placement=placement)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/internals/blocks.py", line 143, in __init__
    f"Wrong number of items passed {len(self.values)}, "
ValueError: Wrong number of items passed 7, placement implies 1
Traceback (most recent call last):
  File "/usr/local/lib/python3.7/site-packages/pandas/core/indexes/base.py", line 3080, in get_loc
    return self._engine.get_loc(casted_key)
  File "pandas/_libs/index.pyx", line 70, in pandas._libs.index.IndexEngine.get_loc
  File "pandas/_libs/index.pyx", line 101, in pandas._libs.index.IndexEngine.get_loc
  File "pandas/_libs/hashtable_class_helper.pxi", line 4554, in pandas._libs.hashtable.PyObjectHashTable.get_item
  File "pandas/_libs/hashtable_class_helper.pxi", line 4562, in pandas._libs.hashtable.PyObjectHashTable.get_item
KeyError: 'images'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/usr/local/lib/python3.7/site-packages/pandas/core/generic.py", line 3826, in _set_item
    loc = self._info_axis.get_loc(key)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/indexes/base.py", line 3082, in get_loc
    raise KeyError(key) from err
KeyError: 'images'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/local/bin/summary_report_pid.py", line 465, in <module>
    main()
  File "/usr/local/bin/summary_report_pid.py", line 458, in main
    final_individual_dataframe = include_images(input_folder, summary_df)
  File "/usr/local/bin/summary_report_pid.py", line 135, in include_images
    summary_df['images'] = summary_df.apply(lambda x: image_finder(x, sample_folder), axis=1)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/frame.py", line 3163, in __setitem__
    self._set_item(key, value)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/frame.py", line 3243, in _set_item
    NDFrame._set_item(self, key, value)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/generic.py", line 3829, in _set_item
    self._mgr.insert(len(self._info_axis), key, value)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/internals/managers.py", line 1203, in insert
    block = make_block(values=value, ndim=self.ndim, placement=slice(loc, loc + 1))
  File "/usr/local/lib/python3.7/site-packages/pandas/core/internals/blocks.py", line 2742, in make_block
    return klass(values, ndim=ndim, placement=placement)
  File "/usr/local/lib/python3.7/site-packages/pandas/core/internals/blocks.py", line 143, in __init__
    f"Wrong number of items passed {len(self.values)}, "
ValueError: Wrong number of items passed 7, placement implies 1

---------------------------------------

ERROR in Script plasmidID on or near line 1089; exiting with status 1
MESSAGE:

See /scratch/bi/SRVCNM1155_20240614_WGSPANAEROBICUS01_svaldezate_S/ANALYSIS/20240704_ANALYSIS11_PLASMIDID/logs/plasmidID.log for more information.
command:
summary_report_pid.py -i /scratch/bi/SRVCNM1155_20240614_WGSPANAEROBICUS01_svaldezate_S/ANALYSIS/20240704_ANALYSIS11_PLASMIDID/NO_GROUP/20240349 -g

---------------------------------------
```

This happens because **PlasmidID expects that plasmids were found for all samples** that were analysed. However, **this is not the case most of the times**; it is pretty common that one or more samples have no plasmids found.

When plasmids are found for a sample, a subfolder called `images` is created within the folder of the sample stored within `NO_GROUP`. This subfolder stores several .png and .conf files. When no plasmids are found for a sample, this `images` subfolder is not created.

Since the `_02_summary_table.sh` script relies on this `images` subfolder to create the final report, it will break if there are any samples for which this subfolder does not exist, displaying the error message that was shown above.

To solve this issue, you should check all the .log files from PlasmidID and find out for which samples no plasmids were found. After that, you can follow this approach, but feel free to do something else if it works better:
1. Copy the folders of the samples for which plasmids were found.
2. Create a new folder apart from `NO_GROUP`, and paste these folders within this new folder.
3. Run `_02_summary_table.sh` using this new folder, which stores only the subfolders of the samples for which plasmids were found, as input. You should not get an error message this time.