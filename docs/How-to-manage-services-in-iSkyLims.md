# How to manage services in iSkyLims

In order to take a service before starting the data analysis, you should firstly have credential for [iSkyLims](https://iskylims.isciii.es/) as well as for the HPC. Use your personal credentials to log in iSkyLims (version 3.0.0) and then click on **Bioinformatics unit: analysis requests**.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/01.iskylims_login.PNG)

On section *Recorded services*, all the new services will be displayed, sorted by the *Service Number* and the researcher who is requesting.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/02.Recorded_services.PNG)

To get more details of the service request, go to the tab **MANAGE SERVICES** and select ***PENDING SERVICES***. A new window, called **Ongoing services** with the tabs ***Services lists*** and ***My services*** will be showed. Under ***Services lists***, click on tab ***Recorded*** and have a look of request description by clicking on the **Service ID**. 

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/03.Manage_services_Recorded.PNG) 

A new window (**Service information**) displays the following:
- general information about the request such as the *Username* of the researcher submitting the request, *Service state*, the date the request was created or even files uploaded which are needed for the data analysis (i.e. accession numbers of reference sequences). 
- ***Requested Services***, containing the services the user has selected for the data analysis.
- ***New Resolutio*n**, with the Service ID following the convention used at the BU-ISCIII.
- ***Services Notes***, containing comments made by the user that may be useful for the data analysis. 
- ***On going resolutions***, by default, it is empty but it will include all the resolutions created for the data analysis. 
- ***Resoultions***, similar to the previous one.
- ***Samples***, the samples the user selected fo the data analysis. 
 
![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/04.Recorded_Service_Information.PNG) 

Before taking the service and creating a new resolution for the current service (by clicking om **Add Reolution**), it is a a good practice to check the notes the researcher incldued in the request and double-check they match the selection of services they made on the iSkyLims interface.

In the service displayed in the images, the user selected `Host genome removal`, `De novo assembly contigs' alignment to database` and `Viral: Detection and characterization of viral genomes within metagenomic data` as ***Requested Services***. However the first selection seems to be incorrect according to the descriptions from ***Services Notes***. An email should be sent to the user to confirm the selection before start the service, if needed. In addition, the user requested to perform *de novo* assembly in th notes but they did not selected `Viral: Genomic reconstruction, variant calling and de novo assembly`. 

It is strongly recommended to modify any detail in the service request before creating any new resolution. In order to do that, log in iSkyLims using `bioinfoadm` credentials, then go to https://iskylims.isciii.es/admin/ and scrowl down to the section **DRYLAB** and click on ***Services***

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/05.Django_admin_iskylims.PNG)

Click on the Service ID to be modified and select the correct options among the list of ***AvailableService***. You may need to select the proper option in ***Service Status***.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/06.Django_admin_change_service.PNG)

You may want to double-check those new changes in the service in the iSkyLims interface. The same proceeding could be used to change other details in a service (i.e. modification of services for a second resolution of the same service). 

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/07.Change_service_on_iskylims.PNG)

Sometimes, samples may need to be addded/removed from the list of data to be processed. This modification is made on ***Samples*** tab. 

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/08.Adding_samples_iSkyLims.PNG)

In order to add new samples, press buttom **Add Samples** and then use the field ``Search``to find the samples to be added (i.e. using the Run ID), select them and then press **Submit**. A new window will show the ID of the samples included in the services. 

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/09.Adding_new_samples_iSkyLims.PNG)

In case of removing samples from the list, choose the Ids of interest in *Select Samples* and then press **Delete Samples**. An alternative to select several samples in a row, you can clik on one and then select the next samples by dragging the black square of the selection rectangule.     

Once the details of the service request have been hecked (and corrected, if needed), create a *new resolution* for that service by clicking **Add Resolution**. A new window will be display the following fields:

- *Estimated resolution date*, enter the turnaround time (TAT) expected for the results to be available for the user.
- *Service acronym*, Some of the most frequently acronyms used at the BU-ISCII are GENOME (followed by the name of the virus) or SARSCOV2 for reference mapping for viruses, VIRAL-DISCOVERY for metagenomics analysis, ASSEMBLY for bateria. In order to select the proper one, check the service request as well as the notes in the *Service Information*. Once the acronym has been selected for the service, we need to assign a number for the current service. For this, we need to go to `/data/bi/services_and_colaborations/CENTRO/AREA/` and search for the last service with the same acronym. For instance, if we have received a service from a Virology lab at the National Center of Microbiology consisting in analysing metagenomics data using Pikavirus, we should go to `/data/bi/services_and_colaborations/CNM/virology/` and search for all the services containing **VIRAL-DISCOVERY** in their name by using `ls | grep "VIRAL-DISCOVERY"`. The screen will display all the services with that acronym followed by a number. Find the last service and use the corresponding next number for the new service (i.e **VIRAL-DISCOVERY666**). The acronym and the specific number must be added in this field.     
- *Assigned user*, the bioinformatician responsible for the analysis of this service (aka yourself).
- *Resolution description*, any information you consider is worth adding to the resolution.   

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/10.Create_First_Resolution_new_service_iSkyLims.PNG)

Once all the fileds have been filled in press **Submit**. The main *Service Information* window will be updated with the Acronym Name (following the convention`SRVRequesting-Area_Date_Service-AcronymXX_researcher_X`), the service state, the *On going resolutions* and *Resolutions*. Make a note of the *Resolution name* (i.e SRVAREAXXX.X) as it is needed for next steps in the service. 

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/11.Service_Info_First_Resolution_iSkyLims.PNG)

You may want to use **On hold** and **In process** depending on the service needs.

There may be times where the user wants to either re-analyse or use different tools on the same data, so that they will request (usually by email) new analysis for the same service. A *new resolution* can be created from the *Service information* window by clicking **Add Resolution**. Modify the service options if needed and proceeed in the same way as explained before.  

Check that new resolution (i.e. `SRVAREAXXX.2`) has been created succesfully on **Ongoing services**, and **My services**. Alternativelly you can click on the service ID and have a look of the details of resolutions available for the same service.

![image](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/12.Several_Resolutions_iSkyLims.PNG)

