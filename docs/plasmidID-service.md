# Plasmid ID service

PlasmidID is a mapping-based, assembly-assisted plasmid identification tool that analyzes and gives graphic solution for plasmid identification.

PlasmidID is a computational pipeline implemented in BASH that maps Illumina reads over plasmid database sequences. The k-mer filtered, most covered sequences are clustered by identity to avoid redundancy and the longest are used as scaffold for plasmid reconstruction. Reads are assembled and annotated by automatic and specific annotation. All information generated from mapping, assembly, annotation and local alignment analyses is gathered and accurately represented in a circular image which allow user to determine plasmidic composition in any bacterial sample.

The software can be accesed via [github](https://github.com/BU-ISCIII/plasmidID) and its manual in its [wiki](https://github.com/BU-ISCIII/plasmidID/wiki)

## Step-by-step guide

## Dependencies

This service depends on `assembly` services as one of th plasmidID inputs is the assembly of the requested samles for analysis. So before you start this service you need to follow the steps in [assembly service](./Assembly-service.md)

### 1. Create the service using [buisciii-tools](https://github.com/BU-ISCIII/buisciii-tools)

- Create the service from the local terminal using iskylims Resolution ID (e.g. `bu-isciii --log-file SRVCNMXXX.X.tool.log new-service SRVCNMXXX.X`). The option `--log-file` will save a log for tracking purposes in a specific location. This option should be used every time the BU-ISCIII tool is used for the service. For instance, you may want to name the log as `SRVCNMXXX.X.new-service.log` if the function you are using is `new-service`. In other cases in which the tool has different options (i.e `scratch`, `bioinfo-doc`), you may want to use the name of the specific function you are about to use to save the log (i.e. `SRVCNMXXX.X.service_to_scratch.log` for tool `scratch` if you transfer data from service to scratch or `SRVCNMXXX.X.delivery.log` for `bioinfo-doc` if you are about to deliver the results). Type `Y` so the folder is not created (it's already created for the assembly step) and only the new plasmidid template is copied.
- A folder will be created in `services_and_colaborations/CENTRE/SERVICE_TYPE` with the full name of the service.
- Go to the recently created folder. Check that the number of reading files matches the number of samples that was specified in the service in iSkyLIMS (the number of files must be number of samples x 2 if they are paired, since there is a file of forward readings and one of reverse readings)

```Bash
cd SRVXXX_YYYYMMDD_XXX_researcheruser_S/RAW
ls -l *.fastq.gz | wc -l
```

- If everything is alright, move to `ANALYSIS` folder and execute `lablog` (This lablog might be named after the name of the template e.g. `lablog_plasmidid`).
- This first lablog will rename the main ANALYSIS folder (`DATE_ANALYSIS01`) to the current date and create folder `00-reads` with symlinks to the fastq files in `RAW`.
- Finally, move the folder to the computing resource using `bu-isciii --log-file SRVCNMXXX.X.tool.log scratch --direction service_to_scratch SRVCNMXXX.X`. Use the specific option you are using to name the log (i.e. `SRVCNMXXX.X.service_to_scratch.log`).
- From now on, all the analysis must be executed from the folder located in scratch.

### 2. Analysis starts

- Execute the lablog inside `DATE_ANALYSIS01_SERVICE`. This lablog will create the following files:
  - `_01_plasmidID.sh`: this script will run plasmidID for the samples on the samples_id.txt file.
  - `_02_summary_table.sh`: this script will collect the results for all samples and consolidate all the information in one report file.

> Make sure you check for any module load or conda env you need to active before running the scripts.

- Run scripts in order.
- Check the logs created in the logs file for any error before running the next script and debug any problem you could find.

### 3. Results validation

- If you are unfamiliar with plasmidID results check its output documentation in bu-isciii tools template reports and its [results interpretation](https://github.com/BU-ISCIII/plasmidID/wiki/Understanding-the-image%3A-track-by-track) manual.
- In order to check the succesful completion of the pipeline you need to check:
  - PlasmidID log for each sample, here you can see if there is any sample for which no plasmid has been found, which you should report in the researcher email.
  - `NO_GROUP/SAMPLE_NAME/images`: check this folder and open some images so you can see if some plasmids have been correctly found. You can check [this manual](https://github.com/BU-ISCIII/plasmidID/wiki/How-to-chose-the-right-plasmids) for this task.
  - Finally if you have more than one sample you must check the summary results file created in the root of `NO_GROUP` folder.

### 4. Finish and deliver service

Once we've checked everything we need to go to the `RESULTS` folder and run the `lablog_plasmidid_results` that will create a deliver folder with the most useful results for the researcher.

Always check that the files are correctly copied/proper links created to the deliver folder.

If everything is correct and all the files have the expected content, we can proceed with the bu-isciii tools finish module:

```Bash
bu-isciii --log-file SRVCNMXXX.X.finish.log finish SRVCNMXXX.X
```

Once someone has reviewed the service and given the ok, you can deliver the service. Remember to move to your workstation for this:

```Bash
bu-isciii --log-file SRVCNMXXX.X.service-info.log bioinfo-doc SRVCNMXXX.X
# select service-info
bu-isciii --log-file SRVCNMXXX.X.delivery.log bioinfo-doc SRVCNMXXX.X
# select delivery
```


### mRNAseq report template (TEAM STANDUP)

Once you have finished the analysis and analyzed the results you need to report to the rest of the members. Use this template to do it:

**SRVXXXYYYY_YYYYMMDD_PLASMIDIDZZZ_researcher_S**

- Servicio: [service name]
- Carrera: [WGS run name / WGS project name]
- Muestras: [number of samples analyzed in the service]
- Solicitante: [name of the person who requested the service]
- Laboratorio: [laboratory alias of the requester]
- Estado: [status of the service]
- Cantidad de lecturas: [the median amount of reads]
- Calidad de las lecturas: [quality of reads]
- Especie identificada (revisión kmerfinder): p.e Neisseria gonorrhoeae
- Revisión general de plásmidos identificados:
- Incidencias muestras individuales (todo lo anterior + Quast):
  - P.e mal ensamblado
  - muestras sin plásmidos identificados
