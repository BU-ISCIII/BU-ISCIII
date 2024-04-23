## Description
Our production and development machines are located in two environments Bioinfo01 server and production IT environment (this is "a matter of concern" due to network restrictions among different environments).

Bionfo01 is a HP ProLiant DL385 G7 server, with the next characteristics:
- 2 AMD Opteron  16 cores a 2.2GHz y 16 MB cache L3 (32 cores) 
- 128 GB RAM 4Rx4 PC3-8500R-7 RDIMM (8 modules of 16GB)
- 8 TB (6.37 TB) RAID 5. HP 1TB SAS 6Gbps MDL 7.2K 6,4cm hot plug.
- 7 TB network resource storage QNAP.
- 2 TB network resource storage (VNX).
- 2 network cards con 2 Gigabit Ethernet  ports each (HP NC382i Dual Port Multifunction Gigabit)
- 1 network card con 2 ports 10 Gigabit Ethernet

Bioinfo01 is virtualized using VMware vSphere 6 (ESXi 6), which allows the deployment of multiple virtual machines with different OS  at the same time without interference. The server is inside ISCIII's Vcenter which comprises all institution datacenters. BU-ISCIII Datacenter is identified as "Bioinformatica" and the ESXI server as "Bioinfo01".

Virtual machines are grouped in several pool resources which allows segmentation/reservation of hardware resources (CPU and RAM) and manage prioritization. 

Bioinfo01 is organized in the next pool resources:

- **Front-end machines:** VMs offered to ISCIII researchers with bioinformatic software for performing their own analysis. This is a service included in the Unit service portfolio.
- **Back-end machines:** VMs used by BU-ISCIII members and students.
- **Curso_ngs:** VMs used in the training courses organized by the Unit in the ISCIII.
- **Laboratory:** testing VMs.
- **Services:** VMs configured as service machines, currently panoramix (former samba file server, deprecated, now Neptuno) and amnesix (backup file server).

### Front-end machines

VMs owned by ISCIII researchers with bioinformatic software to performe their own analysis. This machines are created by ISCIII IT department and maintained both by IT and BU-ISCIII groups.

Machines are requested using BU-ISCIII service forms (iSkyLIMS) and once evaluated the real researcher need BU-ISCIII request the creation of the VM to the IT service using GLPI issue tracker including the next information:

- Operating system.
- RAM.
- Number of cores.
- Disk storage capacity.
- Custom ports to open if needed.
- ISCIII user to grant access permission (by default bioinfoadm, smonzon and icuesta users are included with root permissions)
- Software to include in the installation. Software will be evaluated and agreed upon between BU-ISCIII and the requesting user.

### Back-end machines
VMs used by BU-ISCIII members and students. This virtual machines include mainly machines used for development of new applications/software.

#### VMs for development:
- **Barometrix**: VM for Galaxy develop. 
- **Flavia**: VM for iSkyLIMS develop. Currently is used as "preproduction machine"
- **Cuadrix**: VM for iSkyLIMS develop.
- **Praline**: VM for PlasmidID web development.
#### Other VMs:
- **Falbala**: ubuntu machine with pymol software installed.
- **Gelatina**: basic centos machine for various purposes.

### Curso_ngs

VMs used in the training courses organized by the Unit in the ISCIII. Each time a course is going to start, a request to the IT deparment (GLPI) is made for the virtual machines deployment.

VMs are named **ideafixXX** with the characteristics determined for the course, by default:

- OS: Centos 6.10 
- RAM: 16 gb
- Cores: 2
- Users: alumno, profesor, bioinfoadm
- Password: Ngs%45
- Root permission granted.
- Software: nextflow, singularity, igv

### Services
Services machines are mainly used as linux filesystem servers. This machines are created and maintained by our IT department, with some tasks delegated to us.

- **Panoramix**: linux filesystem implementing a SAMBA/CIFS server. **DEPRECATED: NOW THIS IS ALLOCATED IN NEPTUNO**
    - SO: centos 6.10
    - User: bioinfoadm
    - Permissions: 
        - write permissions for managing samba shared folders 
        - Samba server restart.
    - Main shared folders:
        - bioinfo_doc: main documentation folder for BU-ISCIII (See [[bioinfo_doc]])
        - courseNGS: shared folder for courses documentation to the students.
        - NGSGenomica: shared folder with Genomica Unit.
        - master2017: shared folder for Master students.
        - Shared folders for service delivery (See how to deliver a service in [[Delivery methods]]).
- **Amnesix**: VM for backup tasks.

### Production machines
This machines are deployed in the IT production environment:

- **Neptuno**: linux filesystem implementing a SAMBA/CIFS server. 
    - SO: centos 6.10
    - User: bioinfoadm
    - Password: ask for it! 
    - Permissions: 
        - write permissions for managing samba shared folders 
        - Samba server restart.
    - Main shared folders:
        - bioinfo_doc: main documentation folder for BU-ISCIII (See [[bioinfo_doc]])
        - courseNGS: shared folder for courses documentation to the students.
        - NGSGenomica: shared folder with Genomica Unit.
        - master2017: shared folder for Master students.
        - Shared folders for service delivery (See how to deliver a service in [[Delivery methods]]).
