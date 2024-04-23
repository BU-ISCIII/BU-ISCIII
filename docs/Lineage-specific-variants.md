# Lineage specific variants

## Do the analysis in the HPC

### 1. Go to the folder with the results

```
cd /data/bi/research/20230922_LINEAGE_VARIANTS_OUTBREAKINFO_SV
mkdir $(date '+%Y%m%d')_variants
cd $(date '+%Y%m%d')_variants
```

### 2. Prepare the folder

1. Create a `lablog` file with this content:

```
# module load GDAL

# conda activate R-4.3.2

mkdir logs

echo "srun --partition short_idx --cpus-per-task 1 --mem 35000M --time 01:00:00 --output logs/LINEAGE_VARIANTS.%j.log Rscript ../get_mutations_outbreak_info.R --lineage_list Lineages_Mutations.xlsx --output_folder ${PWD} &" > _01_run_get_mutations.sh
```
 
2. Copy the excell file with the list of lineages from which obtain the variants to the $(date '+%Y%m%d')_variants folder.

### 3. Run the pipeline

```
module load GDAL
conda activate R-4.3.2
bash lablog
bash _01_run_get_mutations.sh
```

:warning: If for any reason it gives an authentication error from outbreak.info, open R in the terminal and authenticate with GISAID credentials as indiated in their manual:

https://outbreak-info.github.io/R-outbreak-info/

## Do it in barbarroja

If it breaks for any reason in the HPC, run it in barbarroja.

### 1. Connect to barbarroja

```
ssh bioinfoadm@10.22.140.224
```

### 2. Go to the folder with the results

```
cd /data/bi/research/20230922_LINEAGE_VARIANTS_OUTBREAKINFO_SV
mkdir $(date '+%Y%m%d')_variants
cd $(date '+%Y%m%d')_variants
```

### 3. Prepare the folder

1. Create a `lablog` file with this content:

```
mkdir logs

echo "nohup Rscript ../get_mutations_outbreak_info.R --lineage_list Lineages_Mutations.xlsx --output_folder ${PWD} > logs/VARIANTS.log 2>&1 &" > _01_run_get_mutations.sh
```
 
2. Copy the excell file with the list of lineages from which obtain the variants to the $(date '+%Y%m%d')_variants folder.

### 4. Run the pipeline

```
bash lablog
bash _01_run_get_mutations.sh
```

## To do

We have to check how exactly outbreak.info works because I don’t know if it need to be updated to run it or if it automatically acess the real time DB through an API.

## How did I install everything in barbarroja in 2024/01/24

### 1. Update R to latest version

```
sudo apt update -qq
sudo apt install --no-install-recommends software-properties-common dirmngr
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
sudo apt install --no-install-recommends r-base
```

:warning: I had to give 777 permissions to `/usr/local/lib/R/site-library/`

### 2. Install devtools:

System libraries

```
sudo apt-get install libcurl4-openssl-dev libssl-dev libxml2-dev libharfbuzz-dev libfribidi-dev
```

R libraries

```
install.packages("devtools")
```

### 3. Install outbreak.info library

System libraries

```
sudo apt-get install libudunits2-dev libgdal-dev
```

R libraries

```
devtools::install_github("outbreak-info/R-outbreak-info")
```

### 4. Install other libraries for the script

System libraries

```
sudo apt-get install 
```

R libraries

```
install.packages("optparse")
install.packages("readxl")
install.packages("writexl")
install.packages("tidyverse")
```