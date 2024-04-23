# BU-ISCIII documentation for SARS-2 service 28/12/2021

# Table of Contents

1. [A user at iSkyLIMS is required](#user)
2. [Login iSkyLIMS ](#login)
3. [Select "Dry Lab"](#drylab)
4. [Select "Manage services" and then  "Pending Services"](#pending)
5. [In terminal](#terminal) 
* 5.1 [Folder Name](#foldername)

* 5.2 [Inside folder](#insidefolder)

* 5.3 [Create folders](#createfolder)

* 5.4 [Inside RAW ](#raw)

* 5.5 [Inside ANALYSIS](#analysis)

* 5.6 [Inside ANALYSIS01](#analysis01)

* 5.7 [Inside ANALYSIS02_MET ](#analysis01met)

* 5.8 [Inside ANALYSIS01,  into `_pangolin` folder ](#pangolin)

* 5.9 [ANALYSIS01,  into `_variants_table` folder ](#variants)

### 1. A user at iSkyLIMS is required<a name="user"></a>

[iSkyLIMS](https://iskylims.isciii.es/) 

### 2. Login  at iSkyLIMS  with user<a name="login"></a>

[iSkyLIMS](https://iskylims.isciii.es/) 

### 3. Select "Dry Lab"  <a name="drylab"></a>

### 4. Select "Manage services" and then  "Pending Services"<a name="pending"></a>

Inside "Pending Services" there are 3 categories: 

1.  Recorded: Services that have been requested

2.  Queued: Services that have been accepted but not started yet.

3.  In progress: Services that are being currently done

In this case the SARS-2 services indicate: 

* Service Request ID: **SRVCNM...**

* Service Requested by: **icasas**

Select "Add resolution" inside the service that is going to be performed

Fill in: 

* **Estimated resolution date**

* **Acronym Name**: 

To get the acronym name, connect to HPC asterix and go to: 
 `/data/bi/services_and_colaborations/CNM/virologia/` and search the last SARS-2 service by **SARSCOV...**


* **ResolutionAsignedUser**: The current user

* **Resolution notes**: Only if needed

* **Action to be done for the service**: **"Service is accepted"** 

Now go backwards twice at the webpage level and refresh. At this point the **Folder Name** is now available. 

### 5. In terminal <a name="terminal"></a>

#### 0. Using mount_processing_data.sh to mount the repository bi_xtutatis and connect to ssh using: 
- sudo sshfs -o allow_other,default_permissions -p XXXXX tu.email@direccion_portutatis.es:/data/bi /data/bi

- ssh -p 32122 tu.email@direccion_portutatis.es

#### 1. Create the folder with the **Folder Name** that appears in iSkyLIMS  <a name="foldername"></a>  

It must be created in the path `/data/bi/services_and_colaborations/CNM/virologia/`. 

**e.g**:  `mkdir SRVCNM548_20211227_SARSCOV274_icasas_S`

#### 2. Go inside the folder just created  <a name="insidefolder"></a>  


**e.g**: `cd SRVCNM548_20211227_SARSCOV274_icasas_S`

#### 3. Create folders  <a name="createfolder"></a>  


`mkdir ANALYSIS RAW REFERENCES TMP DOC RESULTS `

IMPORTANT  BEFORE RUNNING THE SERVICE update references

Copy lablog in references folder /data/bi/scratch_tmp/bi/references/pangolin/
After that, go to DOC folder and change the path to point to the folder with the updated date when the DDBB was last updated , which you just did. The singularity image must be also updated to the version.
#### 4. Go inside the RAW_NC folder  <a name="raw"></a>  


`cd RAW_NC/`


Here, create a symbolic link to the reads. To know the folder to which the link must be made look in iSkyLIMS  **"Projects requested in the service"**.

**WARNING** Take a look at the folder where the reads are: 

**e.g**: `ls /srv/fastq_repo/NextSeq_GEN_344_20211223_ICasas/`

If the sample identifier starts by number this will correspond to **Human host**. If this identifier is not numeric, the **host is different from human**. An email must be sent to the researcher requesting the service, to ask them which is the host if it is not specified in iSkyLIMS section **Notes included in the service request** . Inside de repository where fastq files could be mixed with different samples like influenza and SARS-CoV-2. This can be mitigated corroborated with the samples id from the researcher.

If the host is known, perform the symbolic link.

**e.g**:  `ln -s /srv/fastq_repo/NextSeq_GEN_344_20211223_ICasas/* . `

WARNING double check because there might be links performed to several locations and then several symbolic links must be performed.

#### 5. Go inside the ANALYSIS folder  <a name="analysis"></a>  

`cd ..` 

`cd ANALYSIS/`

List all the previous services and select one that is of the **same host** 

`ls /data/bi/services_and_colaborations/CNM/virologia/*SARSCOV2*/ANALYSIS/`

Copy here the lablog of the selected previous analysis and the samples_ref.txt: 

**e.g**: `cp /data/bi/services_and_colaborations/CNM/virologia/SRVCNM530_20211207_SARSCOV271_icasas_S/ANALYSIS/lablog .`
**e.g**: `cp /data/bi/services_and_colaborations/CNM/virologia/SRVCNM530_20211207_SARSCOV271_icasas_S/ANALYSIS/samples_ref.txt .`

Review the lablog and the files just copied: 

`cat lablog`

BEFORE THIS LABLOG DO:

conda activate nf-core-viralrecon-1.2.0dev


In this case the lablog should look something like this: 

```
cat samples_ref.txt | cut -f1 > samples_id.txt
mkdir -p 00-reads
cp /data/bi/pipelines/config_files/hpc_slurm.config ../DOC/
cat samples_ref.txt | cut -f3 | sort -u | while read in; do echo ${in^^}; done > host_list.tmp
i=1; cat host_list.tmp | while read in
do
    FOLDER_NAME=$(echo $(date '+%Y%m%d')_ANALYSIS0${i}_AMPLICONS_${in})
    mkdir ${FOLDER_NAME}
    cp create_summary_report.sh ${FOLDER_NAME}/
    cp percentajeNs.py ${FOLDER_NAME}/
    grep -i ${in} samples_ref.txt | cut -f1,2 > ${FOLDER_NAME}/samples_ref.txt
    echo "ln -s ../00-reads ." > ${FOLDER_NAME}/lablog
    printf "ln -s ../samples_id.txt .\n\n" >> ${FOLDER_NAME}/lablog
    echo "#module load Nextflow singularity" >> ${FOLDER_NAME}/lablog
    printf 'scratch_dir=$(echo $PWD | sed "s/\/data\/bi\/scratch_tmp/\/scratch/g")\n\n' >> ${FOLDER_NAME}/lablog
    cut -f2 ${FOLDER_NAME}/samples_ref.txt | sort -u | while read ref
    do
        echo "sample,fastq_1,fastq_2" > ${FOLDER_NAME}/samplesheet_${ref}.csv
        grep -i ${ref} ${FOLDER_NAME}/samples_ref.txt | while read samples
        do
            arr=($samples); echo "${arr[0]},00-reads/${arr[0]}_R1.fastq.gz,00-reads/${arr[0]}_R2.fastq.gz" >> ${FOLDER_NAME}/samplesheet_${ref}.csv
        done
        REF_FASTA=$(find /data/bi/references/virus/ -name ${ref}.fasta)
        REF_GFF=$(find /data/bi/references/virus/ -name ${ref}.gff)
        echo "cat <<EOF > ${ref}_viralrecon.sbatch" >> ${FOLDER_NAME}/lablog
        echo "#!/bin/sh" >> ${FOLDER_NAME}/lablog
        echo "#SBATCH --ntasks 1" >> ${FOLDER_NAME}/lablog
        echo "#SBATCH --cpus-per-task 2" >> ${FOLDER_NAME}/lablog
        echo "#SBATCH --mem 4G" >> ${FOLDER_NAME}/lablog
        echo "#SBATCH --time 2:00:00" >> ${FOLDER_NAME}/lablog
        echo "#SBATCH --partition middle_idx" >> ${FOLDER_NAME}/lablog
        echo "#SBATCH --output ${ref}_$(date '+%Y%m%d')_viralrecon.log" >> ${FOLDER_NAME}/lablog
        printf "#SBATCH --chdir \$scratch_dir\n\n" >> ${FOLDER_NAME}/lablog
        printf 'export NXF_OPTS="-Xms500M -Xmx4G"\n\n' >> ${FOLDER_NAME}/lablog
        echo "nextflow run /scratch/bi/pipelines/nf-core-viralrecon-2.4.1/workflow/main.nf \\\\" >> ${FOLDER_NAME}/lablog
        echo "          -c ../../DOC/hpc_slurm.config \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --input samplesheet_${ref}.csv \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --outdir ${ref}_$(date '+%Y%m%d')_viralrecon_mapping \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --platform illumina \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --protocol amplicon \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --variant_caller ivar \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --consensus_caller bcftools \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --fasta ${REF_FASTA} \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --gff ${REF_GFF} \\\\" >> ${FOLDER_NAME}/lablog
        echo '          --primer_bed "/data/bi/references/virus/2019-nCoV/amplicons/NC_045512.2/V4/nCoV-2019.artic.V4.scheme.bed" \\' >> ${FOLDER_NAME}/lablog
        echo '          --kraken2_db "/data/bi/references/eukaria/homo_sapiens/hg38/UCSC/kraken2/kraken2_human.tar.gz" \\' >> ${FOLDER_NAME}/lablog
        echo "          --skip_assembly \\\\" >> ${FOLDER_NAME}/lablog
        echo "          --skip_nextclade" >> ${FOLDER_NAME}/lablog
        printf "EOF\n\n" >> ${FOLDER_NAME}/lablog
        printf "echo 'sbatch ${ref}_viralrecon.sbatch' > _01_run_${ref}_viralrecon.sh\n\n" >> ${FOLDER_NAME}/lablog
    done
  echo "#conda activate nf-core-viralrecon-1.2.0dev" >> ${FOLDER_NAME}/lablog
  echo 'echo "python ./percentajeNs.py */variants/ivar/consensus/bcftools %Ns.tab"  > _02_run_percentage_Ns.sh' >> ${FOLDER_NAME}/lablog
  printf 'echo "bash create_summary_report.sh" > _03_create_stats_table.sh\n\n' >> ${FOLDER_NAME}/lablog
    i=$((i+1))
done
rm host_list.tmp
mkdir $(date '+%Y%m%d')_ANALYSIS0${i}_MAG
rm create_summary_report.sh
rm percentajeNs.py
cd 00-reads; cat ../samples_id.txt | xargs -I % echo "ln -s ../../RAW_NC/%_*R1*.fastq.gz %_R1.fastq.gz" | bash; cat ../samples_id.txt | xargs -I % echo "ln -s ../../RAW_NC/%_*R2*.fastq.gz %_R2.fastq.gz" | bash; cd -
CURRENT_DIR=$(pwd | cut -d '/' -f1,2,3,4,5,6,7)
echo "srun --partition short_idx rsync -rlv ${CURRENT_DIR} /scratch/bi" > _01_copy_folder.sh
```
This lablog does (among other things): (TO CHANGE)
1. Create the samples_id.txt file

2. Creates the 00-reads folder

3. Creates the folder for the analysis (date '+%Y%m%d')_ANALYSIS01_AMPLICONS_HUMAN with current date

4. Created the folder for the meta-genomics analysis (date '+%Y%m%d')_ANALYSIS02_MET with current date

5. Performs a symbolic link and renames the with a shorter name the samples

After checking the lablog is correct, run the lablog: 

`bash lablog`

Check everything is correct by doing: 

`ls`

`ls 00-reads/`

`wc -l samples_id.txt `

#### 6. Go inside the ANALYSIS01 folder <a name="analysis01"></a>  

**e.g**: `cd 20211227_ANALYSIS01_AMPLICONS_HUMAN/` 

Check the automatically created lablog as before: 

`cat lablog`

In this case the lablog should look something like this:

```
ln -s ../00-reads .
ln -s ../samples_id.txt .

#module load Nextflow singularity
scratch_dir=$(echo $PWD | sed "s/\/data\/bi\/scratch_tmp/\/scratch/g")

cat <<EOF > NC_045512.2_viralrecon.sbatch
#!/bin/sh
#SBATCH --ntasks 1
#SBATCH --cpus-per-task 2
#SBATCH --mem 4G
#SBATCH --time 2:00:00
#SBATCH --partition middle_idx
#SBATCH --output NC_045512.2_20220418_viralrecon.log
#SBATCH --chdir $scratch_dir

export NXF_OPTS="-Xms500M -Xmx4G"

nextflow run /scratch/bi/pipelines/nf-core-viralrecon-2.4.1/workflow/main.nf \\
          -c ../../DOC/hpc_slurm.config \\
          --input samplesheet_NC_045512.2.csv \\
          --outdir NC_045512.2_20220418_viralrecon_mapping \\
          --platform illumina \\
          --protocol amplicon \\
          --variant_caller ivar \\
          --consensus_caller bcftools \\
          --fasta /data/bi/references/virus/2019-nCoV/genome/NC_045512.2.fasta \\
          --gff /data/bi/references/virus/2019-nCoV/genes/NC_045512.2.gff \\
          --primer_bed "/data/bi/references/virus/2019-nCoV/amplicons/NC_045512.2/V4/nCoV-2019.artic.V4.scheme.bed" \\
          --kraken2_db "/data/bi/references/eukaria/homo_sapiens/hg38/UCSC/kraken2/kraken2_human.tar.gz" \\
          --skip_assembly \\
          --skip_nextclade
EOF

echo 'sbatch NC_045512.2_viralrecon.sbatch' > _01_run_NC_045512.2_viralrecon.sh

#conda activate nf-core-viralrecon-1.2.0dev

echo "python ./percentajeNs.py */variants/ivar/consensus/bcftools %Ns.tab"  > _02_run_percentage_Ns.sh

echo "bash create_summary_report.sh" > _03_create_stats_table.sh
```

This lablog does the following (among other things): (TO CHANGE)

1. Remove the # and activate module load Nextflow and  singularity

2. Symbolic link to the folder 00-reads 

3. Symbolic link to samples_id.txt 

4. Create samplesheet.csv with header sample,fastq_1,fastq_2 

5. Copy from samples_id.txt to samplesheet.csv the names of the samples in specific format 

6. Create executable _01_viralrecon_mapping.sh  with all the specifics to run nextflow

7. **Commented** Create excutable only when **mapping** must be done 

8.  **Commented**  Run the _01_viralrecon_mapping.sh executable with nohup

9. Copy `create_summary_report.sh` 

10. Copy `percentajeNs.py`

11. Create executable `_03_create_summary.sh` that to create summary report 

12. Create the folder for Pangolin with current date: (date '+%Y%m%d')_pangolin

13. Create the folder for variants analysis with current date: (date '+%Y%m%d')_variants_table

After checking the lablog is correct, run the lablog: 

`bash lablog`

**WARNING** Double check that the automatically created NC_045512.2_viralrecon.sbatch points to /scratch/bi as follows: 

e.g #SBATCH --chdir /scratch/bi/SRVCNM614_20220401_SARSCOV282_pacopozo_S_test/ANALYSIS/20220418_ANALYSIS01_AMPLICONS_HUMAN

At this point the service folder created at /data/bi/services_and_colaborations/CNM/virology/EXAMPLE_FOLDER must be copied (USING rsync -rlv) to /data/bi/scratch_tmp/bi. 

e.g rsync -rlvc /data/bi/services_and_colaborations/CNM/virology/SRVCNM614_20220401_SARSCOV282_pacopozo_S_test/ANALYSIS/20220418_ANALYSIS01_AMPLICONS_HUMAN /data/bi/scratch_tmp/bi" 

Check at the destination that all the symbolic links are still valid at the folder  00-reads. 

Now you can run the command:

`bash _01_run_NC_045512.2_viralrecon.sh`


If everything is OK nextflow will start. Wait a moment to see if does not give any error. 

Once nextflow is running go again into iSkyLIMS and mark the service as **"In progress"** 


To check if nextflow is running corretly you can perform the following: 

`tail -f *_viralrecon.log`

`squeue`

Before computing the stats in scratch_tmp do the following to avoid duplicates in the long table and avoid affecting the stats. 

mv ./variants_long_table.csv ./variants_long_table_dups.csv

head -n1 ./variants_long_table_dups.csv > ./variants_long_table.csv

grep -v 'SAMPLE' ./variants_long_table_dups.csv | sort -u >> ./variants_long_table.csv


#### 7. Go inside the folder  20220418_ANALYSIS01_MAG <a name="analysis01met"></a> 


`cd ..`

**e.g**: `cd 20220418_ANALYSIS01_MAG/`

This folder serves to save the metagenomic information. In here copy the lablog from the 20220418_ANALYSIS01_MAG folder of the path from which we copied the previous lablog: 


**e.g**:  `cp /data/bi/services_and_colaborations/CNM/virologia/SRVCNM530_20211207_SARSCOV271_icasas_S/ANALYSIS/20220418_ANALYSIS01_MAG/lablog .`


Check the lablog as before: 

`cat lablog`

In this case the lablog should look something like this: 

```
ln -s ../00-reads .
ln -s ../samples_id.txt .

#module load Nextflow
#module load singularity

scratch_dir=$(echo $PWD | sed "s/\/data\/bi\/scratch_tmp/\/scratch/g")

cat <<EOF > mag.sbatch
#!/bin/sh
#SBATCH --ntasks 1
#SBATCH --cpus-per-task 2
#SBATCH --mem 4G
#SBATCH --time 2:00:00
#SBATCH --partition middle_idx
#SBATCH --output $(date '+%Y%m%d')_mag.log
#SBATCH --chdir $scratch_dir

export NXF_OPTS="-Xms500M -Xmx4G"

nextflow run /data/bi/pipelines/mag/main.nf \\
          -c /data/bi/pipelines/config_files/hpc_slurm_mag.config \\
          --reads '00-reads/*_R{1,2}.fastq.gz' \\
          --outdir $(date '+%Y%m%d')_mag \\
          --kraken2_db /data/bi/references/kraken/minikraken_8GB_20200312.tgz \\
          --skip_busco --skip_spades --skip_spadeshybrid --skip_megahit \\
          -resume
EOF

echo "sbatch mag.sbatch" > _01_run_mag.sh

mkdir -p 99-stats
```

This lablog does the following (among other things): (TO CHANGE)

1. Symbolic link to 00-reads

2. Create the executable **_01_mag.sh** to run nextflow with the all the specific configuration

3. **Commented** Run the _01_mag.sh executable with nohup

After checking the lablog is correct, run the lablog: 

`bash lablog`


**WARNING** If when running an error is shown concerning reading/writing, run it again. It is already configured to **resume** 

The progress can be check doing: 

` tail -f 20211227_mag01.log `

#### 8. Go inside the folder  ANALYSIS01,  into `_pangolin` folder <a name="pangolin"></a>

THIS STEP IS NO LONGER NEEDED

**DO NOT PERFORM THIS STEP UNTIL THE MAIN NEXTFLOW HAS FINISHED**

**e.g**: `cd 20211227_pangolin/`

In here copy the lablog from the ANALYSIS01  `_pangolin` folder of the path from which we copied the previous lablog:  

`cp /data/bi/services_and_colaborations/CNM/virologia/SRVCNM530_20211207_SARSCOV271_icasas_S/ANALYSIS/20211209_ANALYSIS01_AMPLICONS_HUMAN/20211209_pangolin/lablog .`

Check the lablog as before: 

`cat lablog`

In this case the lablog should look something like this:  

```
#conda activate pangolin
#pangolin --update
#pangolin --update-data
cat ../*_viralrecon_mapping/variants/varscan2/consensus/*.AF0.80.consensus.masked.fa > human_masked.fasta
echo "qsub -V -b y -j y -cwd -N PANGOLIN -q all.q pangolin human_masked.fasta" > _01_pangolin.sh
pangolin -v  >> version.log
pangolin -pv >> version.log
pangolin -dv >> version.log
#cat ../samples_id.txt | while read in; do grep -P "${in}," lineage_report.csv; done | tr ',' '\t' | cut -f2`
#cat ../samples_id.txt | while read in; do grep -P "${in}," lineage_report.csv; done | tr ',' '\t' | cut -f2 | sort -u | grep -v 'None' | sed ':a;N;$!ba;s/\n/, /g'
```

This lablog does the following: 

1. Remove # to activate the conda pangolin environment

2. Update pangolin version 

3. Update pangolin verison 

**WARNING** Check pangolin release in [github](https://github.com/cov-lineages/pangolin/releases). If there is a new release different from the installed a new environment and a new installation of the new relase of Pangolin are required. 

In case of different **percentages of N**  use: 

`pangolin --help` 

The option to change it is `--max-ambig` (By default 0.3 /1) 

After checking the lablog is correct, run the lablog: 

`bash lablog`

#### 9. Go inside the folder  ANALYSIS01,  into `_variants_table` folder <a name="variants"></a>
 
`cd ..`

**e.g**: `cd 20211227_variants_table/`

In here copy the lablog from the ANALYSIS01 `_variants_table` folder of the path from which we copied the previous lablog:  

**e.g**: `cp /data/bi/services_and_colaborations/CNM/virologia/SRVCNM530_20211207_SARSCOV271_icasas_S/ANALYSIS/20211209_ANALYSIS01_AMPLICONS_HUMAN/20211209_variants_table/lablog .`


Check the lablog as before: 

`cat lablog`

In this case the lablog should look something like this: 

```
#conda activate nf-core-viralrecon-1.2.0dev
cat ../samples_id.txt | while read in; do ln -s ../*_viralrecon_mapping/variants/varscan2/${in}.AF0.80.vcf.gz* .; done
#remove samples that didn't finish
VCF_FILES=$(ls *.gz | tr '\n' ' ')
echo "qsub -V -b y -j y -cwd -N BCFTOOLS_MERGE -q all.q bcftools merge -m none $VCF_FILES-Ov -o all_samples_merged.vcf" > _01_bcftools_merge.sh
echo "bcftools query -H -f '%POS\t%REF\t%ALT\t[%DP\t]\n' all_samples_merged.vcf > all_variants.table" > _02_bcftools_query.sh
echo "sed -i 's/\# //g' all_variants.table" > _03_parse_table.sh
echo "sed -i 's/\[[0-9]*\]//g' all_variants.table" >> _03_parse_table.sh

#Create template for SnpSift annotation
cat ../*_viralrecon_mapping/variants/varscan2/snpeff/*.AF0.80.snpSift.table.txt | sort -u > snpSift_template.txt
cat snpSift_template.txt | cut -f2,3,4,5,8,13,14 > snpSift_template_filtered.txt
cat ../*_pangolin/lineage_report.csv | tr ',' '\t' | cut -f 1,2 | sort -u | head -n -1 > samples_lineage.txt
echo "Rscript parser.R" > _04_create_tables.sh
cp /data/bi/services_and_colaborations/CNM/virologia/SRVCNM349_20210308_SARSCOV236_icasas_S/ANALYSIS/20210308_ANALYSIS01_AMPLICONS_HUMAN/20210324_variants_table/parser.R .
```

This lablog does the following: 

1.  Remove # to  activate   nf-core-viralrecon-1.2.0dev environment

2.  To remove samples that didn't finish

3. Create the executable  _01_bcftools_merge.sh that will call bcftools

4. Create the executable _02_bcftools_query.sh that will perform a  query in bcftools 

5.  Create executable _03_parse_table.sh 

6.  Create template for SnpSift annotation  order the files in ascending order 

7. Select the columns that contain important information from `snpSift_template.txt` into  `snpSift_template_filtered.txt`

8. Create ` samples_lineage.txt`from pangolin `lineage_report.csv` 

9. Create executable _04_create_tables.sh that will call the R script to create long and wide table 

10. Copy R script from previous service that will be used to create long table 
 



 



```python

```