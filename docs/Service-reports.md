# Service reports

Once a service has been complete, a report has to be generated both for the receiver and for our records. Follow the next steps to create a standarised report:

1. Move to the service folder in bioinfo_doc and create a new folder inside the result folder named `YYYYMMDD_entregaXX`, where `YYYYMMDD` is the date and `XX` is the number od the resolution.

2. Move into this new folder and copy the following files:

```
cp /processing_Data/bioinfo_doc/services/2020/SRVIIER215_20200211_TRIO145_bmartinezd_S/result/20200708_entrega02/lablog ./
cp /processing_Data/bioinfo_doc/services/2020/SRVIIER215_20200211_TRIO145_bmartinezd_S/result/20200708_entrega02/service_data_template.json.txt ./
```

3. (Optional) Copy also the pipeline report and/or the email if there is anything there to keep (in case of the example folder, this file is `Entrega02_email.pdf`).
4. Inside `lablog`, change the sarvice code and optional pdf name if used.
5. Introduce the right metadata of the service in `service_data_template.json.txt`
6. Run the lablog:

```
bash lablog
```

7. Once the files are ready to be delivered, create a tree file to have a log of all the files delivered:

```
tree /processing_Data/bioinformatics/sftp/<sftpfolder>/<serviceID> > <entregaID>.tree
```

Example: `/processing_Data/bioinfo_doc/services/2020/SRVIIER215_20200211_TRIO145_bmartinezd_S/result/20200708_entrega02`
