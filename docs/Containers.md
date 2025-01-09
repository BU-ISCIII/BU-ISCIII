# Introduction

## Operating-system-level virtualization

Operating-system-level virtualization, also known as containerization, refers to an operating system feature in which the kernel allows the existence of multiple isolated user-space instances. Such instances, called containers, may look like real computers from the point of view of programs running in them. A computer program running on an ordinary operating system can see all resources (connected devices, files and folders, network shares, CPU power, quantifiable hardware capabilities) of that computer. However, programs running inside a container can only see the container's contents and devices assigned to the container.

Think about it as the heir of virtual machines, also, It was useful to me thinking about a massive virtualenv mode like python enviroments.

<img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/VMs-and-Containers.jpg" width="70%"/>

## Implementation

There are multiple programs that implement containers, most famous nowadays are Docker and Singularity, and are ones BU-ISCIII is implementing in its workflows.

### Docker vs Singularity

Generally containers have been designed to solve a single primary use case for a **enterprise** (micro-service virtualization). Unfortunately this is not the case for scientific computation were we are interested on the reproducibility of complex software environments. Because of this situation, and although Docker and Singularity are very similar in concept and usage, Singularity has been developed thinking in scientific HPC environments, and provide some useful features, as User NameSpace, in contrast with Docker that works through a daemon owned by root.

<img src="https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/containers-for-science-and-highperformance-computing-41-1024.jpg" width="100%"/>

## Advantages of containers: Singularity

1. **Mobility of Compute**
Mobility of compute is defined as the ability to define, create and maintain a workflow and be confident that the workflow can be executed on different hosts, operating systems (as long as it is Linux) and service providers.
2. **Reproducibility**
The same features which facilitate mobility also facilitate reproducibility. Once a contained workflow has been defined, the container image can be snapshotted, archived, and locked down such that it can be used later and you can be confident that the code within the container has not changed.
3. **User Freedom**
Singularity can give the user the freedom they need to install the applications, versions, and dependencies for their workflows without impacting the system in any way. Users can define their own working environment and literally copy that environment image (single file) to a shared resource, and run their workflow inside that image.
4. **Support on Existing Traditional HPC**
Singularity can run on host Linux distributions from RHEL6 (RHEL5 for versions lower than 2.2) and similar vintages, and the contained images have been tested as far back as Linux 2.2 (approximately 14 years old). Singularity natively supports InfiniBand, Lustre, and works seamlessly with all resource managers (e.g. SLURM, Torque, SGE, etc.) because it works like running any other command on the system. It also has built-in support for MPI and for containers that need to leverage GPU resources.

## Singularity Hub and Docker Hub

There are two main repositories for container download [Docker Hub](https://hub.docker.com/) and [Singularity Hub](https://singularity-hub.org/). Docker Hub is much more complete, and has a lot of different versions of base SO. There is no problem of compatibility between Docker and Singularity, so you can find the docker you need in Docker Hub and use it with Singularity.

## Biocontainers

[Biocontainers](https://biocontainers.pro/) is an Elixir project that builds and mantains docker and singularity images for every software included in bioconda.

You can go to this [webpage](https://depot.galaxyproject.org/singularity/) and search for an image for any software and version include in bioconda. Always use a singularity image when you need a software, this way you won't need to install anything and reproducibility will be more assured.
