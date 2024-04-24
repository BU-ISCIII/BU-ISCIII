### **NEW SERVICE**

In order to take a service and start processing the data the BU-ISCII receives, you should firstly have credential to enter [iSkyLims](https://iskylims.isciii.es/) and to enter the HPC (portutatis.isciii.es). Once you have got the credential, you can now login iSkyLims (version 3.0.0).

![01 iskylims_login](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/b12a5dbf-d027-4157-b4cf-1bdd2d4e9c86)


CLick on **Bioinformatics unit: analysis requests** and the page will display under *Recorded services* the new services sorted by the Service Number and the researcher who requested the service.

 
![02 Recorded_services](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/1e6e8264-fdb5-4ea2-bed2-59f12808f06b)

To get more details of the service request, go to the tab ***MANAGE SERVICES*** and select ***PENDING SERVICES***. A new window with the tabs ***Services lists*** and ***My services*** will show up. Under ***Services lists***, click on tab ***Recorded*** and have a look of the service information by clicking on the service ID for that request. 

![03 Manage_services_Recorded](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/a73ba428-4081-4476-bf15-0af4e41f86b1)


The new window shows the Services the researcher selected in iSkyLims on the leftr side and on the right side the summary of the services under (***New resolution: \*\*\****) and comments that the user included at the time of the request (***Services Notes***) 
 
![04 Recorded_Service_Information](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/8308940e-e6ed-4ae4-9829-ca6b031c566e)



#### **SERVICE REQUEST MOFIDIFICATION**

*  *SERVICE CHANGES*. 

It is a good practise to check the notes the researcher incldued in the request and double check this matches with the selection they made. For example, in the current example the user selected in iSkyLims the services "Host genome removal", "De novo assembly contigs' alignment to database" and "Viral: Detection and characterization of viral genomes within metagenomic data" but "Host genome removal" is incorrect according to their notes. At the time, they did not select the option "Viral: Genomic reconstruction, variant calling and de novo assembly", useed to carry out *De Novo* assembly. All this things can be modified using the iSKylims bioinfoadm credentials and it is strongly recommeded to do it before creating the service resolution. 

Once you have logged as bioinfoadm in iSkyLims (ask for the password if you do not know it), got to https://iskylims.isciii.es/admin/ and scrowl down to the section **DRYLAB** and click on ***Services***
![05 Django_admin_iskylims](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/f28dfb7d-dece-4b4c-b5e1-93cc409d9810)

Click on the the Service ID you want to work on and modify the services by choosing the correct option among the list of *AvailableService* and select the correct option (Recorded for this example) in *Service Status*.
![06 Django_admin_change_service](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/39b8704f-6d98-486e-959b-5a26b908343a)

You can check that the status has been changed on iSkyLims login with bioinfoadm

![07 Change_service_on_iskylims](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/f952ec06-8a68-40e2-9e8f-082443850a11)


The same proceeding could be used for changes in the resolution (once the services has been created) or any other kind of things. 


*  *ADDING/REMOVING NEW SAMPLES* 

There would be times that some samples have not been included or the user added the wrong ones. For this porpuse the request can be modified on the *Samples* tab (on the bottom of the main *Service information* window). 

![08 Adding_samples_iSkyLims](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/05e0b7d9-1910-4c72-9c5b-f864a5e223ef)


- *adding new samples*, when you press buttom *Add Samples* a new window is open and you can use the field *Search* to try to find the samples to add to the list (for instance, using the Run ID and then selection the sample(s) of interest, and then press *Submit*. A new window will show up with the ID of the samples included in the services. Click on *Return to service* to go back to the main *Service informaction* window.  

![09 Adding_new_samples_iSkyLims](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/29c09e01-895f-4ca7-aea3-4b6b05f699f7)


- *removing samples*, in the column *Select Samples*, you can tick on the sample you want to remove from the list or if you want to select several samples, tick one and then select the next samples by dragging the black square on the rectangule. Then press *Delect samples* a new window will show the samples removed from the list.    


#### **CREATING A NEW RESOLUTION**

By default a new service does not have a resolution until this one is created and the field allocated fo it remains empty in the main *Service information* window.
![10 Default_resolution_new_service_iSkyLims](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/1838717c-25d4-49b3-8cb9-7e1491535cfc)

In order to create a new resolution for a service we need to click **Add Resolution** and a new window will be displayed containing the following fields:
- *Estimated resolution date*, enter the turnaround time (TAT) expected for the results to be available for the user.
- *Service acronym*, depending of the service that we are working with the acronym GENOME, VIRAL-DISCOVERY, ASSEMBLY.... (TODO) 

- *Assigned user*, the bioinformatician responsible for the analysis of this service (aka yourself).
- *Resolution description*, any information you consider is worth adding to the resolution, tracking-wise.   

![image](https://github.com/BU-ISCIII/BU-ISCIII/assets/63557623/1febe714-0f79-44a7-b4c3-5af5c4645af5)


Once we are sure the service request matches what the reseracher is expecting, we can create a new resolution by clicking **Add Resolution** 
