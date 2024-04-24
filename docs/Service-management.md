# Introduction

In this section we are going to explain how BU-ISCIII manages service requests. BU-ISCIII provides support to ISCIII users offering a set of services described in our portfolio, which will be described in the next section. Users can ask for a service using our iSkyLIMS application, it is implemented using a web interface and users can access select the service they want to ask, identify the data to analyze and track the status of its request in time.

The service flow begins when a user asks for a new service, the petition is recorded using iSkyLIMS and a notification is sent to the user and to BU-ISCIII responsible. The petition is evaluated and a resolution is created, accepting or denying the service. If accepted, the service is queued until the responsible for the task is available and then passes to in progress state. Finally, when the service is finished a delivery is sent to the researcher and the service passes to Delivered state.

<img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/master/images/service_management.png" width="100%"/>

## States description

### Recorded

BU-ISCIII users can ask for a new service using iSkyLIMS web interface. iSkyLIMS allows to ask for all the services in the Unit portfolio using one of the forms in iSkyLIMS. There they can select the data they want to analyze, the type of service and include any needed information for the analysis. For more information about how iSkyLIMS works please go to [iSkyLIMS wiki](https://github.com/BU-ISCIII/iSkyLIMS/wiki/DRYLAB) in its own repository.

Users can download a request confirmation with all the data they have included in the petition, and a notification of a new service recorded is sent by email to BU-ISCIII responsible and the user.

### Resolution/Queued

Once a new service is recorded, BU-ISCIII members can access to all the service information in iSkyLIMS, and create a new resolution. In this step BU-ISCIII can accept or deny the service.

- A service is denied in case the information provided don't allow the resolution of the petition or in the case the tasks required for its resolution don't fulfill the requirements of the service requested, according to the service portfolio and documentation. Each service is described in the portfolio including expected input and output. If the user don't provide the needed input, of if a different output is expected the service will be denied. In this case, BU-ISCIII will contact the researcher to solve the requirements, or for deeper evaluation of the service maybe changing the solution to a collaboration with the requesting group.

- If a service is accepted, a bunch of information is set:
  - A new service identifier is generated for the service and a new folder is created in bioinfo_doc/services/{YEAR} including request and resolution pdf with all the service information. Identifications have the next structure: SRV(CNM|IIER|UFIEC)000 and folders have this structure SRV(CNM|IIER|UFIEC)_DATE_LABEL_USER_S where
    - SRV(CNM|IIIER|UFIEC)XXX: indicates the requesting center depending on the requesting user affiliation and XXX is a number identifying the service.
    - DATE: is the resolution date.
    - LABEL: is a identification label for better location of the service in the resultant folders and files. For example TRIO01... for exome trio services.
    - USER: isciii requesting user, for example smonzon.
    - S: indicates that this folder belongs to a service (more information in [[bioinformatics]] structure and [[FAQs]])
  - A estimated delivery date is set depending on the Unit workload, number of samples and type of service and a notification is sent by mail to the requesting user.
  - Service state changes to queued until the assigned bioinformatician is available.

**NOTE:** Many resolutions can be added to a service as our experience shows that researchers can ask for additional results in their petitions.

### In progress

Once the assigned bioinformatician to a service stats a queued service iSkyLIMS is used to set the service to In progress state. A new notification is send to the requesting user informing of the change of state.

### Delivered

Finally when the assigned bioinformatician has completed a service a new delivery is created. The delivery is marked in iSkyLIMS but analysis results are sent to the user using one of the [[Delivery methods]] described in other section. Moreover a results report is sent with all analysis information (software versions, pipeline description, results highlight...) depending on the type of analysis requested.

More than one delivery can be added to a service, if more than one resolution has been created. Each resolution has its own delivery in 1:1 relation.
