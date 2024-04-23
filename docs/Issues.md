# 1. I/O erros

We have observed many times jobs failing with no errors in the code, but because I/O errors. These jobs generate a binary `core.*` file up to 20Gb in size inside the job working directory.

HPC administrators do not know the reason for these errors. We are tracking them and our observations suggest that it may be related to java and python highly demanding processes.

Example:
```
[2018-07-31 17:52:49] Beginning TopHat run (v2.1.1)
-----------------------------------------------
[2018-07-31 17:52:49] Checking for Bowtie
                  Bowtie version:        2.2.5.0
Traceback (most recent call last):
  File "./tophat-2.1.1.Linux_x86_64/tophat", line 4107, in <module>
    sys.exit(main())
  File "./tophat-2.1.1.Linux_x86_64/tophat", line 3938, in main
    copy(params.gff_annotation, t_gff)
  File "/processing_Data/bioinformatics/pipelines/miniconda2/lib/python2.7/shutil.py", line 133, in copy
    copyfile(src, dst)
  File "/processing_Data/bioinformatics/pipelines/miniconda2/lib/python2.7/shutil.py", line 98, in copyfile
    copyfileobj(fsrc, fdst)
IOError: [Errno 5] Input/output error
```

# 2. HPC long response times

Sometimes the whole cluster (both access nodes and computing nodes) gets frozen, and the response times in the terminal can delay several minutes the ouput of a simple `ls` command. This may be slowing down running jobs too, but we still need to benchmark them in order to have reliable data about the issue.

HPC administrators do not know the reason for these issues.

This may be caused by bottlenecking while accessing hard drives, and a possible solution would be using file system designetd to work in this kind of scenarios and not a general one. In the following example, taken on 11/09/2018, we can see how this affects our work. With less than 5% on average of computational capacity in use in each node, and even a smaller proportion of RAM in use, operating was impossible. Most of jobs in execution where reading/writing from/into "/processing_Data/bioinformatics/research", and during execution time it took even a minute to navigate inside its subfolders, list files or edit a line in the terminal:

```
[mjuliam@asterix02 ~]$ qhost 
HOSTNAME                ARCH         NCPU  LOAD  MEMTOT  MEMUSE  SWAPTO  SWAPUS
-------------------------------------------------------------------------------
global                  -               -     -       -       -       -       -
obelix01                linux-x64      40  0.44  252.4G    4.2G    8.0G     0.0
obelix02                linux-x64      20  6.06  252.2G    7.3G    8.0G   14.6M
obelix03                linux-x64      20 12.23  252.2G   13.2G    8.0G   16.8M
obelix04                linux-x64      20  4.13  252.2G    6.5G    8.0G   16.3M
obelix05                linux-x64      20  3.91  252.2G    3.7G    8.0G   15.6M
obelix06                linux-x64      20  3.72  252.2G    2.5G    8.0G   16.2M
obelix07                linux-x64      20  6.30  252.2G    7.6G    8.0G   15.8M
obelix08                linux-x64      20  4.08  252.2G    6.4G    8.0G   16.6M
obelix09                linux-x64      20  6.02  252.2G    7.4G    8.0G   15.1M
obelix10                linux-x64      20  2.98  252.2G   10.4G    8.0G   13.6M
obelix11                linux-x64      20  4.00  252.2G    5.2G    8.0G   15.9M
obelix12                linux-x64      20  6.97  252.2G    4.0G    8.0G   15.2M
obelix13                linux-x64      20  2.89  252.2G    2.2G    8.0G   14.9M
obelix14                linux-x64      20  6.39  252.2G    8.8G    8.0G   15.3M
obelix15                linux-x64      20  4.93  252.2G    6.6G    8.0G   16.0M
obelix16                linux-x64      20  7.79  252.2G    7.5G    8.0G   15.5M
```

# 3. Extreme slowdowns of running jobs

Sometimes the same job takes 6h of processing, others days. We are running statistics to try to discover what's happeining here, but most probably is related with #1 and #2. 

Until now, we have discovered it may caused by a bottleneck in the NFS connection, and extremely high meta-data access. Remember that in our cluster, `/home` (personal libraries and scripts), `/opt` (basically all software, including python, java and R) and `/processing_Data/bioinformatics` (all input, throughput and output data) share the same NFS.

This is a real example of the high meta-data access while taken some jobs were running at 12:06 of 31/07/2018, where we can see all drives mounted by NFS, tha all of them shar the same connection and that the meta-data traffic is nearly 45% of traffic:
```
[mjuliam@asterix02 ~]$ df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-root_lv
                      976M  547M  379M  60% /
tmpfs                 2.9G     0  2.9G   0% /dev/shm
/dev/vda1             380M   98M  263M  28% /boot
/dev/mapper/rootvg-backup_lv
                      283M  2.1M  266M   1% /backup
/dev/mapper/rootvg-tmp_lv
                      976M  2.0M  923M   1% /tmp
/dev/mapper/rootvg-usr_lv
                      4.8G  3.6G  960M  80% /usr
/dev/mapper/rootvg-var_lv
                      2.0G  635M  1.2G  35% /var
/dev/mapper/rootvg-varlog_lv
                      976M   73M  852M   8% /var/log
172.21.31.11:/vol/HPC_opt
                       55G   50G  5.3G  91% /opt
172.21.31.11:/vol/HPC_users
                      200G   43G  158G  22% /home
172.21.31.11:/vol/Processing_UI/antibioticos
                       10T  8.5T  1.6T  85% /processing_Data/antibioticos
172.21.31.11:/vol/Processing_UI/microscopia_electronica
                       10T  8.5T  1.6T  85% /processing_Data/microscopia_electronica
172.21.31.11:/vol/Processing_UI/utic
                       10T  8.5T  1.6T  85% /processing_Data/utic
172.21.31.12:/vol/Processing_BI
                       20T   18T  2.0T  90% /processing_Data/bioinformatics
172.21.31.11:/vol/Processing_UI/epidemiologia
                       10T  8.5T  1.6T  85% /processing_Data/epidemiologia
//172.21.30.96/hpc-bioinfo
                      500G  187G  314G  38% /processing_Data/bioinformatics/sftp

[mjuliam@asterix02 ~]$ netstat | grep :nfs
tcp        0      0 172.21.31.56:1001           172.21.31.11:nfs            ESTABLISHED 
tcp        0      0 172.21.31.56:cadlock        172.21.31.12:nfs            ESTABLISHED 

[mjuliam@asterix02 ~]$ nfsstat -c
Client rpc stats:
calls      retrans    authrefrsh
282358238   33028      282369180

Client nfs v3:
null         getattr      setattr      lookup       access       readlink     
0         0% 82198035 29% 1679677   0% 11414798  4% 30968753 10% 56261     0% 
read         write        create       mkdir        symlink      mknod        
93076715 32% 55252292 19% 411667    0% 49195     0% 99170     0% 0         0% 
remove       rmdir        rename       link         readdir      readdirplus  
1251906   0% 84009     0% 208809    0% 24638     0% 8321      0% 3751611   1% 
fsstat       fsinfo       pathconf     commit       
1822359   0% 16        0% 8         0% 0         0% 
```

According to [Singularity documentation](https://singularity.lbl.gov/about): 
> On HPC systems a single image file optimizes the benefits of a shared parallel file system! There is a single metadata lookup for the image itself, and the subsequent IO is all directed to the storage servers themselves. Compare this to the massive amount of metadata IO that would be required if the container’s root file system was in a directory structure. It is not uncommon for large Python jobs to DDOS (distributed denial of service) a parallel meta-data server for minutes! 

So, using Singularity containers locally inside the computing nodes and working in the local `/scratch` with nextflow may help reducing the meta-data bottleneck and the overusage of `/opt` for every software, leaving the NFS connection only for reading input files and writing results.

Still, a local installation of software in each node instad in a shared `/opt` might be enough for less demanding pipelines and manual analysis.

Both solutions have to be impleneted and tested before exposing them to the HPC admins, or they may prevent us from doing it until the developement environment is ready.

HPC administrators do not know the reason for these issues.
Example:
```
Log:
[2018-08-01 16:44:58] Reconstituting reference FASTA file from Bowtie index
  Executing: /opt/bowtie/bowtie2-2.2.5/bowtie2-inspect ../../REFERENCES/Homo_sapiens.GRCh38.dna.toplevel.fa > C-EXP-2/tmp/Homo_sapiens.GRCh38.dna.toplevel.fa.fa
[2018-08-01 18:13:53] Generating SAM header for ../../REFERENCES/Homo_sapiens.GRCh38.dna.toplevel.fa
[2018-08-01 18:14:12] Reading known junctions from GTF file
[2018-08-01 18:26:19] Preparing reads
         left reads: min. length=50, max. length=75, 27593767 kept reads (16209 discarded)
        right reads: min. length=50, max. length=75, 27265099 kept reads (344877 discarded)
[2018-08-01 19:03:16] Building transcriptome data files ../../REFERENCES/Homo_sapiens.GRCh38.93
[2018-08-01 19:20:42] Building Bowtie index from Homo_sapiens.GRCh38.93.fa
[2018-08-01 19:41:38] Mapping left_kept_reads to transcriptome Homo_sapiens.GRCh38.93 with Bowtie2
date: Thu Aug  2 12:41:22 CEST 2018. Still running.
```
Another slow but without stops:
```
[2018-08-01 15:51:39] Beginning TopHat run (v2.1.1)
-----------------------------------------------    
[2018-08-01 15:51:39] Checking for Bowtie          
                  Bowtie version:        2.2.5.0   
[2018-08-01 18:07:40] Checking for Bowtie index files (genome)..
[2018-08-01 18:07:40] Checking for reference FASTA file         
        Warning: Could not find FASTA file ../../REFERENCES/Homo_sapiens.GRCh38.dna.toplevel.fa.fa
[2018-08-01 18:07:40] Reconstituting reference FASTA file from Bowtie index                       
  Executing: /opt/bowtie/bowtie2-2.2.5/bowtie2-inspect ../../REFERENCES/Homo_sapiens.GRCh38.dna.toplevel.fa > C-DIF-2/tmp/Homo_sapiens.GRCh38.dna.toplevel.fa.fa
[2018-08-01 18:57:53] Generating SAM header for ../../REFERENCES/Homo_sapiens.GRCh38.dna.toplevel.fa                                                            
[2018-08-01 18:58:20] Reading known junctions from GTF file                                                                                                     
[2018-08-01 18:58:56] Preparing reads                                                                                                                           
         left reads: min. length=50, max. length=75, 30068207 kept reads (17579 discarded)                                                                      
        right reads: min. length=50, max. length=75, 29631545 kept reads (454241 discarded)                                                                     
[2018-08-01 19:11:54] Building transcriptome data files ../../REFERENCES/Homo_sapiens.GRCh38.93                                                                 
[2018-08-01 19:34:01] Building Bowtie index from Homo_sapiens.GRCh38.93.fa                                                                                      
[2018-08-01 19:54:52] Mapping left_kept_reads to transcriptome Homo_sapiens.GRCh38.93 with Bowtie2
[2018-08-01 20:15:04] Mapping right_kept_reads to transcriptome Homo_sapiens.GRCh38.93 with Bowtie2
[2018-08-01 20:36:10] Resuming TopHat pipeline with unmapped reads
[2018-08-01 20:36:11] Mapping left_kept_reads.m2g_um to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2
[2018-08-01 20:41:37] Mapping left_kept_reads.m2g_um_seg1 to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2 (1/3)
[2018-08-01 20:42:04] Mapping left_kept_reads.m2g_um_seg2 to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2 (2/3)
[2018-08-01 20:42:34] Mapping left_kept_reads.m2g_um_seg3 to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2 (3/3)
[2018-08-01 20:42:51] Mapping right_kept_reads.m2g_um to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2
[2018-08-01 20:48:15] Mapping right_kept_reads.m2g_um_seg1 to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2 (1/3)
[2018-08-01 20:48:29] Mapping right_kept_reads.m2g_um_seg2 to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2 (2/3)
[2018-08-01 20:48:45] Mapping right_kept_reads.m2g_um_seg3 to genome Homo_sapiens.GRCh38.dna.toplevel.fa with Bowtie2 (3/3)
[2018-08-01 20:48:58] Searching for junctions via segment mapping
[2018-08-01 21:17:51] Retrieving sequences for splices
[2018-08-01 21:39:45] Indexing splices
Building a SMALL index
[2018-08-01 21:40:08] Mapping left_kept_reads.m2g_um_seg1 to genome segment_juncs with Bowtie2 (1/3)
[2018-08-01 21:40:14] Mapping left_kept_reads.m2g_um_seg2 to genome segment_juncs with Bowtie2 (2/3)
[2018-08-01 21:40:21] Mapping left_kept_reads.m2g_um_seg3 to genome segment_juncs with Bowtie2 (3/3)
[2018-08-01 21:40:27] Joining segment hits
[2018-08-01 22:02:08] Mapping right_kept_reads.m2g_um_seg1 to genome segment_juncs with Bowtie2 (1/3)
[2018-08-01 22:02:14] Mapping right_kept_reads.m2g_um_seg2 to genome segment_juncs with Bowtie2 (2/3)
[2018-08-01 22:02:21] Mapping right_kept_reads.m2g_um_seg3 to genome segment_juncs with Bowtie2 (3/3)
[2018-08-01 22:02:26] Joining segment hits
[2018-08-01 22:24:02] Reporting output tracks
-----------------------------------------------
[2018-08-01 23:09:42] A summary of the alignment counts can be found in C-DIF-2/align_summary.txt
[2018-08-01 23:09:42] Run complete: 07:18:03 elapsed
```

Documentation:
* [How to Display NFS Server and Client Statistics](https://docs.oracle.com/cd/E19253-01/816-4555/netmonitor-12/index.html)
* [Parallel distributed file systems](https://wr.informatik.uni-hamburg.de/_media/research/talks/2016/2016-03-03-parallel_distributed_file_systems.pdf)
* [Diagnosing Parallel I/O Bottlenecks in HPC Applications](https://sdm.lbl.gov/students/sc17-harrington-summary.pdf)
* [Parallel file systems for linux clusters](https://es.slideshare.net/RaheemUnnisa1/parallel-file-system-for-linux-clusters)
* [Using Filesystem Virtualization to Avoid Metadata Bottlenecks](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.185.4044&rep=rep1&type=pdf)
* [Managing Cluster Software Packages](http://www.admin-magazine.com/HPC/Articles/Managing-Cluster-Software-Packages)
* [Setting Up an HPC Cluster](http://www.admin-magazine.com/HPC/Articles/real_world_hpc_setting_up_an_hpc_cluster)

# 4. Old libraries

Centos 6 sucks, we need to update at least to Centos 7. We cannot keep working with system libraries from the 90s...

# 5. Storage

We keep running out of disk space in `/processing_Data/bioinformatics`. We need to optimise the archiving proccess and set a storage expansion scheme for the next years based on our data size historical evolution.

# 6. Fastq files in a dedicate storage

We should have a dedicated storage folder for raw fastq files, so we can avoid copying the input files from the sequencing cabin to `/processing_Data/bioinformatics` for every service (doubling the space used by those files in the cluster).

# 7. SGE parameters do not work

Most of `qsub` parameters do not work properly or do not work at all. We should test them all and make a list of good practices so we can successfully control the resources our jobs use.

HPC admins should also fix the ones giving problems, and create dedicate queues.

# 8. Java heap size ("Fixed")
He observado algo que podríamos llamar curioso respecto a la máquina virtual de java. No lo pongo como incidencia porque no es exactamente algo que haya que "solucionar" sólo una observación por si algún otro usuario lo experimenta o da problemas en algún momento.

Con la nueva configuración de la memoria ram por defecto cada job sólo puede usar 12,5 gb de ram como me indicásteis en la resolución de la incidencia para la configuración del parámetro h_vmem del sge. Es memoria RAM de sobra para los jobs individuales en la mayoría de los casos por lo que parece un valor correcto. 

Sin embargo, en el caso de la máquina virtual de java parece ser que por defecto java calcula en función de la memoria RAM física del nodo (1/4th de la memoria física según javadoc, pero varía un poco según la versión y hay discusiones en los foros asíque lo tomo como orientativo), por lo que calcula estos valores de RAM necesarios para crear la máquina virtual: 
    uintx ErgoHeapSizeLimit                         = 0               {product}           
    uintx HeapSizePerGCThread                       = 87241520        {product}           
    uintx InitialHeapSize                          := 2147483648      {product}           
    uintx LargePageHeapSizeThreshold                = 134217728       {product}           
    uintx MaxHeapSize                              := 32038191104     {product}  

Un mínimo de 2 gb y un máximo de 32gb. De forma que parece ser que necesita tener disponible el valor máximo para crearse la máquina virtual, de forma que si intentas crear una máquina virtual con los valores por defecto escribiendo java sin más en la terminal de un nodo obtengo el siguiente error:

Error occurred during initialization of VM
Could not reserve enough space for object heap
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.

La solución es sencilla se tiene que iniciar la máquina de java con la opción -Xmx con un valor menor a 10g para que funcione, o darle como valor a qsub -l h_vmem=>32g siempre que se utilice java. 

Lo que no sé es si convendría cambiar los parámetros por defecto de java para que se ajuste al tamaño configurado en las colas del sge. 

