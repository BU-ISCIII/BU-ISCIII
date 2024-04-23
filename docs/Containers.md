# Introduction
## Operating-system-level virtualization
Operating-system-level virtualization, also known as containerization, refers to an operating system feature in which the kernel allows the existence of multiple isolated user-space instances. Such instances, called containers, may look like real computers from the point of view of programs running in them. A computer program running on an ordinary operating system can see all resources (connected devices, files and folders, network shares, CPU power, quantifiable hardware capabilities) of that computer. However, programs running inside a container can only see the container's contents and devices assigned to the container.

Think about it as the heir of virtual machines, also, It was useful to me thinking about a massive virtualenv mode like python enviroments.

<img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/master/images/VMs-and-Containers.jpg" width="70%"/>

## Implementation 
There are multiple programs that implement containers, most famous nowadays are Docker and Singularity, and are ones BU-ISCIII is implementing in its workflows.

### Docker vs Singularity
Generally containers have been designed to solve a single primary use case for a **enterprise** (micro-service virtualization). Unfortunately this is not the case for scientific computation were we are interested on the reproducibility of complex software environments. Because of this situation, and although Docker and Singularity are very similar in concept and usage, Singularity has been developed thinking in scientific HPC environments, and provide some useful features, as User NameSpace, in contrast with Docker that works through a daemon owned by root.

<img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/master/images/containers-for-science-and-highperformance-computing-41-1024.jpg" width="100%"/>

## Advantages of containers: Singularity
1. **Mobility of Compute**
Mobility of compute is defined as the ability to define, create and maintain a workflow and be confident that the workflow can be executed on different hosts, operating systems (as long as it is Linux) and service providers.
2. **Reproducibility**
The same features which facilitate mobility also facilitate reproducibility. Once a contained workflow has been defined, the container image can be snapshotted, archived, and locked down such that it can be used later and you can be confident that the code within the container has not changed.
3. **User Freedom**
Singularity can give the user the freedom they need to install the applications, versions, and dependencies for their workflows without impacting the system in any way. Users can define their own working environment and literally copy that environment image (single file) to a shared resource, and run their workflow inside that image.
4. **Support on Existing Traditional HPC**
Singularity can run on host Linux distributions from RHEL6 (RHEL5 for versions lower than 2.2) and similar vintages, and the contained images have been tested as far back as Linux 2.2 (approximately 14 years old). Singularity natively supports InfiniBand, Lustre, and works seamlessly with all resource managers (e.g. SLURM, Torque, SGE, etc.) because it works like running any other command on the system. It also has built-in support for MPI and for containers that need to leverage GPU resources.

## Singularity Hub and Docker Hub.
There are two main repositories for container download [Docker Hub](https://hub.docker.com/) and [Singularity Hub](https://singularity-hub.org/). Docker Hub is much more complete, and has a lot of different versions of base SO. There is no problem of compatibility between Docker and Singularity, so you can find the docker you need in Docker Hub and use it with Singularity.

## Sci-f
Scientific Filesystem (SCIF), an organizational format that supports exposure of executables and metadata for discoverability. The format includes a known filesystem structure, a definition for a set of environment variables describing it, and functions for generation of the variables and interaction with the libraries, metadata, and executables located within.
Although scif is not exclusively for containers, in that a container can provide an encapsulated, reproducible environment, the scientific filesystem works optimally when contained. Containers traditionally have one entrypoint, one environment context, and one set of labels to describe it. A container created with a Scientific Filesystem can expose multiple entry points, each that includes its own environment, metadata, installation steps, tests, files, and a primary executable script. SCIF thus brings internal modularity and programatic accessibility to encapsulated, reproducible environments.
SCIF is natively included in Singularity, understand it as a standard for configuring your containers in order to maximize the goals of **modularity**, **transparency**, and **consistency** needed for scientific reproducibility.
 
# BU-ISCIII implementation of Singularity
In BU-ISCIII we have decided to follow Sci-f specification, using Singularity as main container utility, Singularity-Hub as main container repository and github as version control system.

## Workflow for a new nextflow pipeline.
Here we establish a summary of the steps you need to follow for the creation of a new container using SciF specification using SCIF Builder, and adapting its usage for Nextflow. For a initial guide for practice all this stuff you must go to [nextflow-scif repo](https://github.com/BU-ISCIII/nextflow-scif), there you will find how to perform all this next steps:

1. Create github repo
2. Create Singularity/Docker recipe (example in toy repo)
3. SCIF recipe (example in toy repo and all BU-ISCIII scif recipes can be found in [scif_app_recipes repo](https://github.com/BU-ISCIII/scif_app_recipes))
4. Circleci config file (example in toy repo)
5. Nextflow script (example in toy repo)

<img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/master/images/singularity_hub_flow.png" width="100%"/>

You can also find useful this [slides](https://github.com/BU-ISCIII/BU-ISCIII/blob/master/curso_SeqGenBac_session1.X_NextflowAndSingularity.pdf)

# Links
1. [Singularity docs](https://www.sylabs.io/guides/2.5.1/user-guide/introduction.html)
2. [SlideShare Singularity](https://www.slideshare.net/m31/containers-for-science-and-highperformance-computing)
3. [Sci-f](https://sci-f.github.io/)
4. [Sci-f paper](https://watermark.silverchair.com/giy023.pdf?token=AQECAHi208BE49Ooan9kkhW_Ercy7Dm3ZL_9Cf3qfKAc485ysgAAAdMwggHPBgkqhkiG9w0BBwagggHAMIIBvAIBADCCAbUGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMIfPovNBP4ncYLoiWAgEQgIIBhtOv5EomeAGPUsjV1FrstiUmZgw6KrvsOSzq1hsETUum3ocYXIgyLSTsroYb0Gi8PLK6dXrp8tHwdPEXd7K3z4jJEimS0knRmFdlFDHIJbnOwp4Ws0uP8dpYavYXx_yAjMseFpMarBLH4Mvphlf3uj5IjdbuGeuozb4Rne4JutPlYBbmouzPZejH2AvvepQ22cEOwOdMDXCwG00kTSOWO8Dt_IvxysHut96kudnYSyfxRHTtqzw_4cZuhs9EKYZziHDaxIeVzHc898g_84E1nQ3c2DrSbWscWNqPOd_WFr7M1dGlhG1IpgXP7XirNPNZUTQPCr-7s9XvrzuyIlcEBPfyI6ZGQUTRzVda7fq_pd2UehC3l4eLI37nAIoQJs7IYWZY1VG7VLXmGZ_3mjwmG9dTKA5oB1HMXByxFJ0CZbMuunrfebblSM_Y6VadKFhB-2AseD9QWPyDNFdjtxsZU1Pv9Ib3_6dq1t2gQ08FxvIlo9B1Nc5qCbQuK-ZxRASjPbW0g1t0EA)
5. [Sci-f Recipes](https://sci-f.github.io/tutorial-preview-install#install-scif-in-docker-using-recipe)
6. [Sci-f Commands](https://sci-f.github.io/tutorial-commands)
7. [Sci-f Builder 1](https://vsoch.github.io/2018/scientific-filesystem-builder/)[Sci-f Builder 2](https://sci-f.github.io/builder/)
8. [Sci-f and Snakemake](https://sci-f.github.io/apps/examples/snakemake.scif)
9. [Sci-f and Singularity](https://www.sylabs.io/guides/2.5.1/user-guide/reproducible_scif_apps.html)