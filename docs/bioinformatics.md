# `/data/ucct/bi`

This folder is intended to be used for data analysis and temporal results storage. It is our biggest folder but it is not backed up, so data which is wanted to be stored should be moved to the storage cabin as soon as there is space available there. Intermediate files and duplicates have also to be removed to free space for future analysis.

`/data/ucct/bi` folder is mounted in `/data/ucct/bi/` in the HPC environment and should be in this location in all the machines mounting the resource, easing the use of absolute paths in software config files.

Inside `/bi` you can find the following subfolders:

## 3.1. `/references`

Here are all the reference genomes we usually work with. They are organised in different folders according to their species, purpose and tool of reference.

## 3.2. `/research`

Every research project and its files are located in a folder here. We use the following naming protocol for these folders:

- YYYYMMDD_LABEL_RESEARCHERS_T/C, where:
  - YYYYMMDD is the date of creation in the numerical format year-month-day
  - LABEL is the name of the project
  - RESEARCHERS are the initials of the people involved in it
  - T/C stands for the origin of the project as a Thesis (T) or a collaboration (C).

Each project folder has the same internal structure as the projects in /`services_and_colaborations`, which will be explained in the next section.

## 3.3. `/services_and_colaborations`

Here are the folders of every service and/or collaboration we work on. They are split in folders depending on the institution and department they came from. The folders of the projects are named by this rule:

- SRVCODE_YYYYMMDD_LABEL_OWNER_S/C, where:
  - SRVCODE is a code assigned to the service (“SRV” + institution + number of service)
  - YYYYMMDD is the date of creation in the numerical format year-month-day
  - LABEL is the name of the project
  - OWNER are the initials of the IP who asked for the service
  - S/C stands for  Service (S) or collaboration (C).

> The name will match exactly the name of the documentation folder in [[bioinfo_doc]] shared folder.

The internal structure of each project folder  consist on 6 folders:

- `ANALYSIS`: each analysis (scripts, documentation and results) we make will be in a folder here, named as XX-name where XX is the number of the analysis and name is the name of the analysis.
- `DOC`: any documentation required for the service or talks, posters an publications derived from its results.
- `RAW`: raw input data is copied here while we are working on the service.
- `REFERENCES`: reference sequences and files for the analysis.
- `RESULTS`: one folder per each results submission for the service. They are named as YYYYMMDD_entregaXX, where YYYYMMDD is the date of submition in the numerical format year-month-day, and XX is the number of the submission.This folder will be deprecated soon because this information will be in ```bioinfo_doc/services```.
- `TMP`: temporal files.

## 3.4. `/sftp`

Our SFTP repository where we deposit services results for short periods of time so the owner of the service can access them. There is one folder for each kind of service, an we just copy the project folder from `/service_and_colaborations` (excluding files that are not useful for the final user). For learning how to use and manage this resource please read [[Delivery methods]].

## 3.5. `/tmp`

Temporal files and programs.

## 3.6. `/pipelines`

Pipelines code and executables.

## 3.7. `/scratch/tmp`

`/scratch` folder mounted with read and write permissions.

## 3.8. `/logs`

Logs from the `archive` and `autoclean-sftp` modules from `buisciii-tools` are stored here, classified by year.