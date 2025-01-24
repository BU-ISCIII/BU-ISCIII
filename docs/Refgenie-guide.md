# Refgenie usage guide

Welcome to this guide on how to use **refgenie** for the **storage**, **access** and **management** of **reference resources**! By means of this document, you should end up being able to understand how this software works and how to properly use all its functionalities.

## Introduction

According to the authors, [**refgenie**](https://github.com/refgenie/refgenie) manages **storage**, **access**, and **transfer** of reference genome **resources**. It provides command-line and Python interfaces to **download pre-built reference genome "assets"**, like indexes used by bioinformatics tools. It can also **build assets for custom genome assemblies** or any other resources. Refgenie provides programmatic access to a standard genome folder structure, so software can swap from one genome to another.

### Overview

There is a wide range of data, including assemblies and auxiliar files, that are required by several software packages and that are typically shared among them. Given this, research groups tend to organise these resources in central folders to prevent duplication, but each research group might use different strategies to identify genome resources, so sharing tools across groups can be challenging in this sense. 

Given this, there is the iGenomes project that focuses on creating a web-accessible server where standard, organised reference assemblies are available for download. However, this also has the main drawback that these resources are not scripted, so they cannot be modified in order to include any possible resources of interest, making them limited regarding their usefulness. 

**Refgenie** was developed with the objective of making it possible to share interoperable reference genome assets and other resources by means of a more modular, customisable and user-controlled approach for resources management. Refgenie allows for the building of genome assets (like fasta or gff files), as well as programmatic access to individual resources both remote and local.

Refgenie can organise any files that can be assigned to a particular reference genome assembly, being able to handle any asset type, including annotations and indexes. More importantly, it provides individual and pre-built asset downloads from a server and allows scripted building for custom inputs.

In concrete, refgenie simultaneously provides structure to manually build assets while facilitating modular access to pre-built assets in the same system. Refgenie does this by providing two ways to obtain genome assets (see image):

* An interface for scripted asset "builds," each of which produces structured output for arbitrary genome inputs.
* Web-based access to individual pre-built assets via web interface or application programming interface (API).

>[!NOTE]
>All pre-built assets can be consulted in **http://refgenomes.databio.org/**, which can be accessed via a RESTful API.

![Refgenie schema](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/refgenie-schema.png)

This two-pronged approach enables users to either retrieve or produce identically structured outputs on demand for any genome of interest, including new assemblies, private assemblies, or custom genomes for which a public set of assets cannot exist.

### Installation

Refgenie is a Python package that can be installed directly in the HPC using `pip`, or within a **micromamba environment**. In our case, **we already have a micromamba environment** designed for the usage of refgenie, so please run the following before running any commands from refgenie:

```
micromamba activate refgenie_v0.12.1
```

The [**0.12.1**](https://github.com/refgenie/refgenie/releases/tag/v0.12.1) is, currently, the most recent version of refgenie (released on Nov 4, 2021).

## Glossary

**Refgenie** uses a specific set of **terms** that must be fully understood before trying to learn anything else. These concepts are:

* **Genome**: a particular version of a consensus genomic sequence for an organism, or a certain set of reference resources that are associated (depending on the situation).
* **Asset**: a folder consisting of one or more files related to a specific reference.
* **Asset registry path**: a string used to refer to assets, of the form: `genome/asset:tag`. If using an asset with more than 1 key, additional keys are appended like: `genome/key.asset:tag`.
* **Key**: a string identifier for a particular file or folder contained within an asset. A key is used to retrieve a path with `refgenie seek`.
* **Tag**: a unique string identifier that allows for multiple assets of the same name to co-exist. One common use case for tags is to maintain multiple versions of an asset.

Once you have familiarised yourself with these concepts, let's learn how refgenie can be used for the management of reference resources 😃.

## Basic usage of refgenie

### Asset organisation

>[!WARNING]
>Refgenie was originally designed to store data related to specific genomes, that is why the first existing level for data organisation is "**genome**", followed by "**asset**", "**tag**" and "**key**". However, in BU-ISCIII, we are trying to take full advantage of refgenie's potential, by employing it to store our reference resources in a more organised way.
>
>That's why we have "genomes" that, in reality, correspond to viral families, or to databases related to specific software, such as kraken, kmerfinder or ariba. The data associated to these resources are clearly not related to one specific genome, but multiple genomes, but the way in which refgenie organises resources can be quite useful to keep a better track of our resources.

In order to **keep track of metadata**, refgenie uses a local **YAML** file called the **"genome configuration file"**. In the case of BU-ISCIII, you can find this file as **`genome_config.yaml`**, located in `/data/ucct/bi/references/refgenie`.

In this file, refgenie stores **paths** to individual **resources**, or “**assets**”, each of which **represents one or more files**. One can think of an **asset as a folder of related files tied to a particular reference**. For example, an asset could be an index for a particular tool, or a group of annotation files.

Refgenie assets are referred to using “**asset registry paths**”, which are human-readable asset identifiers. The registry path follows the structure **{genome}/{asset}:{tag}**; a "genome" thus operates as a sort of namespace for a set of assets, which are identified both by asset names as well as by **tags**, allowing refgenie to **manage multiple versions of the same asset**. All files related to a specific tag of an asset can be addressed by means of a unique **key**.

Let's have a look at what this YAML file looks like, and you'll understand better the meaning of "**genome**", "**asset**", "**tag**" and "**key**".

![alt text](https://github.com/BU-ISCIII/BU-ISCIII/blob/main/images/conf-file.png)

This YAML file, as you can see, has several attributes (all required):
* **genome_folder**: Path to parent folder refgenie-managed assets.
* **genome_servers**: URLs to a refgenieserver instances.
* **genomes**: A list of "genomes", **each genome has a list of assets**. Any relative paths in the asset path attributes are considered relative to the genome folder in the config file (or the file itself if not folder path is specified), with the genome name as an intervening path component, e.g. `folder/mm10/indexed_bowtie2`.
* **aliases**: A list of arbitrary strings that can be used to refer to the "genome".
* **tags**: A collection of tags defined for the asset. These can be used mainly to store multiple versions of an asset.
* **default_tag**: A pointer to the tag that is currently defined as the default one.
* **asset_parents**: A list of assets that were used to build the asset in question.
* **asset_children**: A list of assets that required the asset in question for building.
* **seek_keys**: A mapping of names and paths of the specific files within an asset.
* **asset_path**: A path to the asset folder, relative to the genome config file.
* **asset_digest**: A digest of the asset directory (more precisely, of the file contents within one) used to address the asset provenance issues when the assets are pulled or built.

In the previous example, which corresponds to the beginning of refgenie's YAML configuration file, there are some initial fields, like `genome_folder` and `genome_servers`, that were already explained. Then, the file continues with the `genomes` field, which occupies the vast majority of the file itself. 

Within this field, we'll see a list of genomes, each one with their corresponding **assets**, **tags** and **keys**. In the previous example, we can see the first "genome" that can be referred in two different ways: a **unique ID composed of several random letters and numbers** and a more user-friendly **alias** (one "genome" can have one or several aliases).

The **`aliases`** field is the first one that is written in this file when creating a new "genome". Then, the different **assets** that belong to this "genome" are listed. In the case of the previous example, we can notice two assets: `fasta` and `ensembl_gtf`. These assets can have one or different **tags**: in this case, both assets have only one tag, called `default`, which is also the **default tag**. Finally, associated with each tag, there are several files, each one of them identified by a specific **key** (`fasta`, `fai`, `chrom_sizes`, `ensembl_gtf`, `ensembl_tss`, etc.).

>[!IMPORTANT]
>**Therefore, we can notice that refgenie organises the data based on a four-level system:**
>* **GENOME**
>   * **ASSET**
>       * **TAG**
>           * **KEY**
> 
> **The KEY level is NOT mandatory. All genomes must have at least one asset and each asset must have at least one tag, but it is not compulsory to name each file after a specific key. This depends on your interests.**

The information that is stored within the refgenie system is not added into the YAML file manually. **This file is updated automatically by means of specific commands that will be explained in this guide**.

### Data storage

Now we know how refgenie keeps track of all the resources that are of our interest but, how does refgenie store these specific resources? 

Refgenie, apart from the YAML configuration file that is essential for its correct usage, has two folders that are employed for data storage this way:
* Inside `/data/ucct/bi/references/refgenie/`, there is a folder called **`data`** (**CAUTION! This is NOT `/data/` from `/data/ucct/bi/`**). Inside this folder, you'll find several subfolders, each one of them with a weird name composed of random letters and numbers. **Don't be scared!** These are just the unique IDs of the "genomes" that are already tracked by refgenie. **Inside each one of these folders, you'll find as many subfolders as assets are associated to a "genome", and inside each one of these folders, you'll find as many subfolders as tags are associated to your asset.**

>[!TIP]
>The `data` folder is where the original files are stored. You'll therefore have to store manually your resources here. It is very recommendable that, before storing the data, you have a clear idea of how you want to store this data, especially considering that you should adapt to the way that refgenie works. 
>
>As you may tell, the idea is that data is stored following this general structure: **`genome/asset/tag`**. Therefore, before starting to store your data here, please take a few minutes to think the name of your "genome", how many assets it's going to have and how many tags each asset is going to have, and how you're going to name each one of them. This is fundamental since the names that you use for your assets and your tags will be the ones that will be added to the YAML configuration file.

* Inside `/data/ucct/bi/references/refgenie/`, there is a folder called **`alias`**. Inside this folder, you should find as many folders you saw in `data`, but this time named after their corresponding aliases. This way, you can access your data in a more user-friendly way. Just like with the `data` folder, the data is organised in this folder following the structure: **`alias/asset/tag`**. The difference with the `data` folder lies on the fact that no actual files are stored in this `alias` folder; this folder simply contains symbolic links to the actual files that are stored in `data`. This `alias` folder exists, therefore, mainly to help the user find specific files more easily.

### Refgenie command line interface (CLI)

Let's finally get started with the main commands that can be used when running refgenie in our HPC. With the corresponding micromamba environment already activated, we'll find this if we simply run `refgenie`:

```
version: 0.12.1 | refgenconf 0.12.2
usage: refgenie [-h] [--version] [--silent] [--verbosity V] [--logdev] {init,list,listr,pull,build,seek,seekr,add,remove,getseq,tag,id,subscribe,unsubscribe,alias,compare,upgrade,populate,populater} ...

refgenie - reference genome asset manager

positional arguments:
  {init,list,listr,pull,build,seek,seekr,add,remove,getseq,tag,id,subscribe,unsubscribe,alias,compare,upgrade,populate,populater}
    init                Initialize a genome configuration.
    list                List available local assets.
    listr               List available remote assets.
    pull                Download assets.
    build               Build genome assets.
    seek                Get the path to a local asset.
    seekr               Get the path to a remote asset.
    add                 Add local asset to the config file.
    remove              Remove a local asset.
    getseq              Get sequences from a genome.
    tag                 Tag an asset.
    id                  Return the asset digest.
    subscribe           Add a refgenieserver URL to the config.
    unsubscribe         Remove a refgenieserver URL from the config.
    alias               Interact with aliases.
    compare             Compare two genomes.
    upgrade             Upgrade config. This will alter the files on disk.
    populate            Populate registry paths with local paths.
    populater           Populate registry paths with remote paths.

options:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  --silent              Silence logging. Overrides verbosity.
  --verbosity V         Set logging level (1-5 or logging module level name)
  --logdev              Expand content of logging message format.

https://refgenie.databio.org
No command given
```

As you can tell, refgenie has a lot of functionalities that can be used depending on our needs. We'll explain the most useful ones in this guide.

>[!WARNING]
>Whatever function you are running, you **ALWAYS** have to run refgenie **indicating the path of the YAML configuration file**. For example, if your PWD is `/data/ucct/bi/references/refgenie/` (the YAML file is located there), you'll have to run the refgenie function you want to run but adding `-c genome_config.yaml` in the end.

#### 1. List all available remote assets

To have a look at all the remote assets that are available in http://refgenomes.databio.org/, you can run the **`listr`** function:

```
(refgenie_v0.12) $ refgenie listr -c genome_config.yaml 
                                                                                              Remote refgenie assets                                                                                               
                                                                                     Server URL: http://refgenomes.databio.org                                                                                     
┏━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ genome           ┃ assets                                                                                                                                                                                       ┃
┡━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ rCRSd            │ fasta, bowtie2_index, bwa_index, star_index, hisat2_index, bismark_bt2_index                                                                                                                 │
│ hg38             │ fasta, gencode_gtf, refgene_anno, dbsnp, ensembl_gtf, ensembl_rb, suffixerator_index, bwa_index, bowtie2_index, dbnsfp, star_index, fasta_txome, hisat2_index, cellranger_reference,         │
│                  │ bismark_bt2_index, salmon_partial_sa_index, tgMap, salmon_sa_index                                                                                                                           │
│ human_repeats    │ fasta, suffixerator_index, bowtie2_index, bwa_index, star_index, hisat2_index, bismark_bt2_index                                                                                             │
│ mouse_chrM2x     │ fasta, suffixerator_index, bowtie2_index, bwa_index, star_index, hisat2_index, bismark_bt2_index                                                                                             │
│ hg18_cdna        │ fasta, kallisto_index                                                                                                                                                                        │
│ hs38d1           │ fasta, suffixerator_index, bowtie2_index, bwa_index, star_index, hisat2_index, bismark_bt2_index                                                                                             │
│ hg38_cdna        │ fasta, salmon_index, kallisto_index                                                                                                                                                          │
│ rn6_cdna         │ fasta, salmon_index, kallisto_index                                                                                                                                                          │
│ mm10_cdna        │ fasta, salmon_index, kallisto_index                                                                                                                                                          │
│ hg19_cdna        │ fasta, salmon_index, kallisto_index                                                                                                                                                          │
│ hg38_chr22       │ fasta, suffixerator_index, bowtie2_index, bwa_index, star_index, hisat2_index, bismark_bt2_index                                                                                             │
│ hg19             │ fasta, gencode_gtf, refgene_anno, dbsnp, ensembl_gtf, ensembl_rb, suffixerator_index, bowtie2_index, bwa_index, hisat2_index, fasta_txome, star_index, cellranger_reference,                 │
│                  │ bismark_bt2_index, salmon_partial_sa_index, tgMap, salmon_sa_index                                                                                                                           │
│ hg18             │ fasta, gencode_gtf, bowtie2_index, suffixerator_index, bwa_index, hisat2_index, fasta_txome, star_index, cellranger_reference, bismark_bt2_index                                             │
│ human_alu        │ fasta, suffixerator_index, bowtie2_index, bwa_index, hisat2_index, bismark_bt2_index                                                                                                         │
│ human_rDNA       │ fasta, suffixerator_index, bowtie2_index, bwa_index, star_index, hisat2_index, bismark_bt2_index                                                                                             │
│ human_alphasat   │ fasta, suffixerator_index, bowtie2_index, bwa_index, star_index, hisat2_index, bismark_bt2_index                                                                                             │
│ mm10             │ fasta, gencode_gtf, refgene_anno, ensembl_gtf, ensembl_rb, suffixerator_index, bwa_index, bowtie2_index, hisat2_index, star_index, fasta_txome, cellranger_reference, bismark_bt2_index,     │
│                  │ salmon_partial_sa_index, tgMap, salmon_sa_index                                                                                                                                              │
│ rn6              │ fasta, refgene_anno, ensembl_gtf, suffixerator_index, bwa_index, bowtie2_index, hisat2_index, star_index, fasta_txome, bismark_bt2_index, salmon_partial_sa_index, tgMap, salmon_sa_index    │
│ t7               │ fasta, bowtie2_index                                                                                                                                                                         │
│ dm6              │ fasta, gencode_gtf, refgene_anno, ensembl_gtf, bowtie2_index                                                                                                                                 │
│ hg38_noalt_decoy │ fasta, suffixerator_index, bwa_index, bowtie2_index, star_index, hisat2_index, bismark_bt2_index                                                                                             │
│ mm10_primary     │ fasta, bowtie2_index, bwa_index                                                                                                                                                              │
│ hg38_primary     │ fasta, bwa_index, bowtie2_index                                                                                                                                                              │
│ hg38_mm10        │ fasta, bwa_index                                                                                                                                                                             │
└──────────────────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                                               use refgenie listr -g <genome> for more detailed view
```

In the previous table, you can see all the "genomes" that are available in this server and all the assets you can get from it (for example, for the rCRSd genome, you can get fasta files and different indexes). 

Please notice that the previous table offers a general version on all the "genomes" that are stored in the server. If you want to know about the tags and the keys associated to the assets showed in the previous table, you can use the `-g` option from `listr` to get more information in regards to a specific "genome":

```
(refgenie_v0.12) $ refgenie listr -c genome_config.yaml -g rCRSd
                        Remote refgenie assets                        
              Server URL: http://refgenomes.databio.org               
┏━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┓
┃ genome  ┃ asset (seek_keys)                              ┃ tags    ┃
┡━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━┩
│ rCRSd   │ fasta (fasta, fai, chrom_sizes, dir)           │ default │
│ rCRSd   │ bowtie2_index (bowtie2_index, dir)             │ default │
│ rCRSd   │ bwa_index (bwa_index, dir)                     │ default │
│ rCRSd   │ star_index (star_index, dir)                   │ default │
│ rCRSd   │ hisat2_index (hisat2_index, dir)               │ default │
│ rCRSd   │ bismark_bt2_index (bismark_bt2_index, dir)     │ default │
└─────────┴────────────────────────────────────────────────┴─────────┘
```

As you can tell, the rCRSd has several assets, all of them have only one tag called default and each assets has several files, each one identified by a specific key.

>[!WARNING]
>Please remember that keys do not necessarily have to be the actual names of the files, they are just a way to identify them.

You'll see later in this guide how to download resources from the server.

#### 2. List all available local assets

In our case, we already have several local resources already stored within the refgenie system. We can get a general overview of all these local resources by means of the **`list`** function from refgenie:

```
(refgenie_v0.12) $ refgenie list -c genome_config.yaml 
                                                                                               Local refgenie assets                                                                                               
                                                                                Server subscriptions: http://refgenomes.databio.org                                                                                
┏━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ genome              ┃ assets                                                                                                                                                                                    ┃
┡━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ hg38                │ fasta, ensembl_gtf, dbsnp, bwa_index, dbnsfp, refgene_anno                                                                                                                                │
│ adenoviridae        │ fasta, gff, ensembl_rb                                                                                                                                                                    │
│ anelloviridae       │ fasta, gff                                                                                                                                                                                │
│ arenaviridae        │ fasta, gff                                                                                                                                                                                │
│ astroviridae        │ fasta, gff                                                                                                                                                                                │
│ caliciviridae       │ fasta, gff                                                                                                                                                                                │
│ caudovirales        │ fasta, gff                                                                                                                                                                                │
│ circoviridae        │ fasta, gff
| kaiju               │ nr_euk                                                                                                                                                                                    │
│ protozoa            │ Plasmodium_falciparum                                                                                                                                                                     │
│ MLVA                │ primers                                                                                                                                                                                   │
│ centrifuge          │ refseq                                                                                                                                                                                    │
│ bacteria            │ summary                                                                                                                                                                                   │
│ kmerfinder          │ archaea, bacteria, fungi, plasmids, protozoa, typestrain, viral                                                                                                                           │
│ metaphlan           │ bowtie2index                                                                                                                                                                              │
│ mirbase             │ fasta                                                                                                                                                                                     │
│ MLST                │ profiles, fasta, bowtie2                                                                                                                                                                  │
│ MASH                │ msh                                                                                                                                                                                       │
│ MEGAN               │ db                                                                                                                                                                                        │
│ tools               │ tools                                                                                                                                                                                     │
│ resistance          │ ARGANNOT                                                                                                                                                                                  │
│ CanSNPer2           │ data, db, references                                                                                                                                                                      │
│ ariba               │ argannot, card, megares, ncbi, plasmidfinder, resfinder, srst2_argannot, vfdb_core, vfdb_full, virulencefinder                                                                            │
└─────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                                               use refgenie list -g <genome> for more detailed view
```

Just like with the remote assets, you can list more detailed information on a specific "genome" by means of the `-g` option:

```
(refgenie_v0.12) $ refgenie list -c genome_config.yaml -g arenaviridae
                                                                                               Local refgenie assets                                                                                               
                                                                                Server subscriptions: http://refgenomes.databio.org                                                                                
┏━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ genome       ┃ asset (seek_keys)                    ┃ tags                                                                                                                                                      ┃
┡━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ arenaviridae │ fasta (fasta, fai, chrom_sizes, dir) │ AB972430.1, AB972431.1, LT601601.1, LT601602.1, NC_004296.1, NC_004297.1, NC_005080.1, NC_005081.1, NC_006572.1, NC_006573.1, NC_006574.1, NC_006575.1,   │
│              │                                      │ NC_010562.1, NC_010563.1, NC_038366.1, NC_038367.1, NC_077806.1, NC_077807.1, ON381477.1, ON381478.1                                                      │
│ arenaviridae │ gff (gff, dir)                       │ AB972430.1, AB972431.1, LT601601.1, LT601602.1, NC_004296.1, NC_004297.1, NC_005080.1, NC_005081.1, NC_006572.1, NC_006573.1, NC_006574.1, NC_006575.1,   │
│              │                                      │ NC_010562.1, NC_010563.1, NC_038366.1, NC_038367.1, NC_077806.1, NC_077807.1, ON381477.1, ON381478.1                                                      │
└──────────────┴──────────────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

In this example, we have a "genome" (even though it's not a real genome) called `arenaviridae`, which has two assets (`fasta` and `gff`), each one of them with several tags (in this case, each tag corresponds to an NCBI ID for a viral sequence from this family).

>[!WARNING]
>**Refgenie does NOT like it when a tag is only numeric, like 20211017 (it breaks since it does not expect the default tag to be only numeric). Please take this into account when creating tag names, as you'll see later**.

#### 3. Download pre-built remote assets

What if you want to download assets from a genome that is available in http://refgenomes.databio.org/? You can do this by means of the **`pull`** function from refgenie, once you have checked that this asset can indeed be downloaded (by running `listr`).

Let's say you want to download the `hisat2_index` asset from the `hg38` genome that is available in this server. To do so, you'll have to run the `pull` function this way:

```
(refgenie_v0.12) $ refgenie pull hg38/hisat2_index -c genome_config.yaml

Compatible refgenieserver instances: ['http://refgenomes.databio.org']
Downloading URL: http://refgenomes.databio.org/v3/assets/archive/2230c535660fb4774114bfa966a62f823fdb6d21acf138d4/hisat2_index
2230c535660fb4774114bfa966a62f823fdb6d21acf138d4/hisat2_index:default ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100.0% • 3.9/3.9 GB • 30.1 MB/s • 0:00:00
Download complete: /data/ucct/bi/references/refgenie/data/2230c535660fb4774114bfa966a62f823fdb6d21acf138d4/hisat2_index/hisat2_index__default.tgz
Extracting asset tarball: /data/ucct/bi/references/refgenie/data/2230c535660fb4774114bfa966a62f823fdb6d21acf138d4/hisat2_index/hisat2_index__default.tgz
Default tag for '2230c535660fb4774114bfa966a62f823fdb6d21acf138d4/hisat2_index' set to: default
Created alias directories:
 - /data/ucct/bi/references/refgenie/alias/hg38/hisat2_index/default
```

Now, after the download is complete, we can check the YAML configuration file, and we'll see that the new asset has been added in relation to the hg38 genome:

```
hisat2_index:
        tags:
          default:
            asset_path: /data/ucct/bi/references/refgenie/data/2230c535660fb4774114bfa966a62f823fdb6d21acf138d4/hisat2_index/default/
            asset_digest: c77a00bdd0505d1d083c43eaec21debe
            seek_keys:
              hisat2_index: .
        default_tag: default
```

#### 4. Create a new "genome" from scratch

We previously saw how to add remote assets to an already existing genome within the refgenie system. However, what do we have to do if we want to add into this system resources that we already have stored somewhere else and that we want to migrate into refgenie so that these resources start being handled by it?

In order to accomplish this, you'll first have to create the "genome" to which your local resources belong, and once this is done you'll be able to add assets with their corresponding tags.

Let's use an example. Let's say you have a list of all plasmids stored in NCBI, along with their corresponding sequences, and you have this information for several dates, each one being the download date. You also have some plasmid annotation files. You want to store this data in a standardised way and keep track of all the dates in which you have downloaded the sequences from NCBI. Refgenie is ideal for this!

First of all, as was advised previously, you have to think how you're going to organise the original data, taking into account that the `data` folder follows the `genome/asset/tag` structure. You have to think what the names of the genome, the assets and the tags are going to be, depending on your needs, how these resources are gonna be accessed and considering how they are being stored so far.

In this case, we'll use the following strategy to name them:
* **GENOME**: `plasmids`
* **ASSETS**:
  * `annotations`
  * `ddbb`
* **TAGS**:
  * Several, depending on the dates (each tag will start with "v" in order to avoid fully numeric tag names).

That being said, we first have to create the "genome" called `plasmids`. To do so, we'll use the **`alias`** function from refgenie. This function, at the same time, can be used for a few things:

```
(refgenie_v0.12) $ refgenie alias -h -c genome_config.yaml 
usage: refgenie alias [-h] {remove,set,get} ...

Interact with aliases.

positional arguments:
  {remove,set,get}
    remove          Remove aliases.
    set             Set aliases.
    get             Get aliases.

options:
  -h, --help        show this help message and exit
```

The `alias` function can be used to remove aliases from an already existing "genome", it can be used to set an alias for a "genome" or it can be used to get the alias of a certain "genome".

When creating a new "genome", therefore, we'll have to first establish its alias by means of **`refgenie alias set`**. You must indicate which alias and which unique ID you'll use to identify the new "genome". The alias is `plasmids` in this case, and the unique ID can be randomly generated by running `openssl rand -hex 24`:

```
(refgenie_v0.12) $ refgenie alias set -a plasmids -d $(openssl rand -hex 24) -f -c genome_config.yaml 
Set genome alias (61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2: plasmids)
Created alias directories:
 - /data/ucct/bi/references/refgenie/alias/plasmids
```

The new "genome" has already been created, and this can be checked in the end of the YAML configuration file:

```
61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2:
    aliases: 
     - plasmids
config_version: 0.4
```
Plus, a new `plasmids` folder has been created in `alias`.

However, the original data that we want to migrate into the refgenie system has not magically been transfered. In fact, inside of the data folder we won't find any folder called `61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2`.

We have to **manually** create this folder and then, inside of it, one folder per asset. In our case, we'll have to create two folders: `annotations` and `ddbb`. Then, inside each one of these folders, we'll have to create as many subfolders as versions we have of the files. The annotations correspond to one single date, but the NCBI plasmids database was downloaded several times, so we'll have a few tags in this case. Then, simply copy or move carefully your original files into the locations where they should be at. In the end, the `/data/ucct/bi/references/refgenie/data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2` folder will look like this:

```
(refgenie_v0.12) [user@portutatis03 61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2]$ tree
.
├── annotations
│   └── v20200312
│       ├── ALL_Virulence.fasta
│       ├── ARGannot.pID.fasta
│       ├── ARIBA_out.card.fa
│       ├── IncA_C.fasta
│       ├── INCLM.fasta
│       ├── IS_PLASMID_ncbi_prot_90.fasta
│       ├── MOB_ncbi_90.fasta
│       ├── plasmidFinder_17_07_2018.fsa
│       ├── qacED1.fasta
│       ├── vfdb_full.fa
│       ├── virulence_ecoli_eaec_stx_ENT.fasta
│       ├── virulencefinder.fa
│       └── virulence_plasmid.fasta_90
└── ddbb
    ├── v20180501
    │   ├── plasmids_may_2018_unnamed_term.fasta
    │   ├── plasmids_may_2018_unnamed_term.fasta.1.bt2
    │   ├── plasmids_may_2018_unnamed_term.fasta.2.bt2
    │   ├── plasmids_may_2018_unnamed_term.fasta.3.bt2
    │   ├── plasmids_may_2018_unnamed_term.fasta.4.bt2
    │   ├── plasmids_may_2018_unnamed_term.fasta.length
    │   ├── plasmids_may_2018_unnamed_term.fasta.rev.1.bt2
    │   ├── plasmids_may_2018_unnamed_term.fasta.rev.2.bt2
    │   └── plasmids_may_2018_unnamed_term.length
    ├── v20200203
    │   ├── 20200203_plasmids.fasta
    │   ├── 20200203_plasmids.fasta.length
    │   ├── 20200203_plasmids.length
    │   ├── 20200203_plasmids.mash.distances.tab
    │   ├── 20200203_plasmids.txt
    │   └── test_ddbb.csv
    └── v20220419
        ├── plasmids.fna
        ├── plasmids.fna.length
        ├── plasmids.length
        ├── plasmids.mash.distances.tab
        └── plasmids.txt
```

These are the **ORIGINAL** files that are now stored within the `data` folder of the refgenie system. However, if you run `refgenie list`, you'll see that the `plasmids` "genome" still has no assets and therefore no tags associated with them. This is because, even though the original files are already stored within the refgenie `data` folder, we still have to "add" these resources into the system, so that refgenie acknowledges them and keeps track of them. This is what will be explained in the next section.

#### 5. Add local resources into the refgenie system

Let's say you have manually imported your local resources into the `data` folder from refgenie, but bear in mind your resources still cannot be recognised by refgenie. You can check this by running `refgenie list`, as was stated in the previous section. No new assets nor tags will be shown regarding this "genome", and there won't be a folder, named after your "genome"'s alias, in the `alias` folder from refgenie.

Therefore, your data is now stored within refgenie but **we still have to make refgenie be aware of this data and track it**. To do so, we can use the **`refgenie add`** function.

Let's keep using the same example that was used in the previous section, regarding the plasmids resources. We have the original files in `data` but we still have to make refgenie keep track of them. Let's run `refgenie add` for that purpose. This function requires for the user to specify the path where the original files are located, and you must indicate the names of the "genome", the asset you want to include and the tag you want to include. Let's run this command for the `annotations` asset and its `v20200312` tag, following the `genome/asset:tag` structure:

```
(refgenie_v0.12) $ refgenie add plasmids/annotations:v20200312 --path /data/ucct/bi/references/refgenie/data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312 -c genome_config.yaml

Default tag for '61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations' set to: v20200312
Added asset: 61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations:v20200312
```

>[!NOTE]
>If, for some reason, you try to add a tag again, when it was already added before, don't worry if you are asked (for this particular example): `'61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb:v20200203' exists. Do you want to overwrite? [y/N]`. You can say yes, and the tag will be removed, just to be re-written. No other tags from any other genomes will be affected by this.

If we repeat this procedure for all the resources that we had in `data` in relation to this "genome", we should find:
* A new folder in `alias` called after the "genome"'s alias (in this case, `plasmids`).
* Inside this folder, as many subfolders as assets there are and, inside of these, as many subfolders as tags there are for each asset.
* Inside these tag subfolders, symbolic links to all the files that are stored in `data`.

In the previous example, we should find something like this:

```
(refgenie_v0.12) [user@portutatis03 v20200312]$ pwd
/data/ucct/bi/references/refgenie/alias/plasmids/annotations

(refgenie_v0.12) [user@portutatis03 annotations]$ tree
.
└── v20200312
    ├── ALL_Virulence.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/ALL_Virulence.fasta
    ├── ARGannot.pID.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/ARGannot.pID.fasta
    ├── ARIBA_out.card.fa -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/ARIBA_out.card.fa
    ├── IncA_C.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/IncA_C.fasta
    ├── INCLM.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/INCLM.fasta
    ├── IS_PLASMID_ncbi_prot_90.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/IS_PLASMID_ncbi_prot_90.fasta
    ├── MOB_ncbi_90.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/MOB_ncbi_90.fasta
    ├── plasmidFinder_17_07_2018.fsa -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/plasmidFinder_17_07_2018.fsa
    ├── qacED1.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/qacED1.fasta
    ├── vfdb_full.fa -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/vfdb_full.fa
    ├── virulence_ecoli_eaec_stx_ENT.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/virulence_ecoli_eaec_stx_ENT.fasta
    ├── virulencefinder.fa -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/virulencefinder.fa
    └── virulence_plasmid.fasta_90 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/annotations/v20200312/virulence_plasmid.fasta_90

1 directory, 13 files
```

Inside the `annotations` folder, we can find as many subfolders as tags there are (in this case, just one) and, inside this subfolder, several symbolic links, pointing to the original files stored in `data`.

You may now be wondering: **what if I want to add my resources so that each file has a key assigned to it**, so that it can be more easily identified? `Refgenie add` allows the user to assign keys to every file that is being added into the refgenie system. This is what will be shown next.

Let's use another example for this purpose. Within the refgenie system, there is a "genome" called `MLVA`, corresponding to several .txt files containing primers used for Multi-Loci VNTR Analysis. This is what we can see in the pertinent `data` folder:

```
(refgenie_v0.12) [user@portutatis03 1c4b90e01d465a0c85db0bee8150f77ef59842718ce3d615]$ tree
.
└── primers
    └── v20231004
        ├── Anthracis_primer.txt
        ├── Brucella_primers.txt
        ├── Burkholderia_primer.txt
        ├── Coxiella_primer.txt
        ├── Legionella_pneumophila_primers.txt
        └── README

2 directories, 6 files
```

As we can see, there is a single asset called `primers`, which has only one tag and several .txt files associated to that tag. This is what `MLVA` looks like in the YAML configuration file:

```
1c4b90e01d465a0c85db0bee8150f77ef59842718ce3d615:
    aliases: 
     - MLVA
    assets:
      primers:
        tags:
          v20231004:
            asset_path: /data/ucct/bi/references/refgenie/data/1c4b90e01d465a0c85db0bee8150f77ef59842718ce3d615/primers/v20231004
            asset_digest: e2b11e5e8ceb427a45ca92eb7569cf29
            seek_keys:
              Anthracis: Anthracis_primers.txt
              Brucella: Brucella_primers.txt
              Burkholderia: Burkholderia_primers.txt
              Coxiella: Coxiella_primers.txt
              Legionella_pneumophila: Legionella_pneumophila_primers.txt
        default_tag: v20231004
```

In this file, if we check the `seek_keys` field, we can see that each .txt file has an associated key (for example, the file `Anthracis_primers.txt` has the key `Anthracis` assigned to it). How was this done when this tag was added into refgenie? By using another option from the `refgenie add` function, as will be shown next:

```
(refgenie_v0.12) [user@portutatis03 refgenie]$ refgenie add MLVA/primers:v20231004 --path /data/ucct/bi/references/refgenie/data/1c4b90e01d465a0c85db0bee8150f77ef59842718ce3d615/primers/v20231004 -c genome_config.yaml --seek-keys '{"Anthracis": "Anthracis_primers.txt", "Brucella": "Brucella_primers.txt", "Burkholderia": "Burkholderia_primers.txt", "Coxiella": "Coxiella_primers.txt", "Legionella_pneumophila": "Legionella_pneumophila_primers.txt"}'
```

By means of the `seek-keys` argument from this function, you can specify the key that each file, stored in the path that is indicated in `--path`, will have. As we can see, this is indicated by a string representation of a JSON object.

>[!WARNING]
>When using the `seek-keys` argument, it is **very important** that you follow the required structure: **`'{"" : "", "" : "", ...}'`**. Otherwise, the keys will not be correctly assigned to your files.
---
<br>

Apart from `refgenie add`, there is another function offered by refgenie that allows for the inclusion of local resources, even though it has some more constraints in comparison to `refgenie add`. This is the case of **`refgenie build`**.

First of all, `refgenie build` can only add specific assets from a limited list, which are:

```
$ refgenie list

Local recipes: bismark_bt1_index, bismark_bt2_index, bowtie2_index, bwa_index, dbnsfp, ensembl_gtf, ensembl_rb, epilog_index, fasta, feat_annotation, gencode_gtf, hisat2_index, kallisto_index, refgene_anno, salmon_index, star_index, suffixerator_index, tallymer_index
```

Therefore, if your local data cannot be assigned to one of these possible assets, this function cannot be used to add these resources into the refgenie system; you'll have to use `refgenie add`. 

If, on the contrary, you have files that can be associated to one of these possible assets, you may add it into refgenie by means of this function. For example, let's say you have a certain "genome", like `GRCm39`, for which you want to add a fasta file. This can be linked to an asset called `fasta`, which is one of the possible assets that can be managed by `refgenie build`, apart from a specific tag:

```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/635/GCF_000001635.27_GRCm39/GCF_000001635.27_GRCm39_genomic.fna.gz

refgenie build GRCm39/fasta:v20240101 --files fasta=GCF_000001635.27_GRCm39_genomic.fna.gz -c genome_config.yaml
```

In the previous example, we downloaded a specific genome in fasta format, and then added it into refgenie using the `build` function, specifying the file and the YAML configuration file by means of the `--files` and `-c` arguments, respectively. Please notice the `refgenie build` function is also based on the `genome/asset:tag` structure. It is not compulsory to specify a tag; in such case, the tag name will be `default`.

>[!WARNING]
>When running `refgenie build`, the files we want to add **MUST be compressed**. That's why, in the previous example, the .fna file that we are adding is compressed. Otherwise, `refgenie build` won't work.

When building assets **with this function**, we have to take into account that s**ome assets cannot be built without having already built other required assets**. For example, in order to build the `bowtie2_index` asset, you must already have a fasta asset managed by refgenie. This requirement can be checked by means of the `-q` argument of `refgenie build`:

```
$ refgenie build hg38/bowtie2_index -q

'bowtie2_index' recipe requirements:
- assets:
    fasta (fasta asset for genome); default: fasta
```

Taking all this information into account, simply use either `refgenie add` or `refgenie build` depending on your personal needs and which function you feel more comfortable with.

#### 6. Know the unique ID of a certain "genome"

Sometimes, it can be useful to find out the unique ID of a certain "genome", especially if you need to modify or add something into the `data` folder and you do not remember (perfectly understandable 🤷‍♂️) the ID of the "genome" you're interested in. This is what the **`refgenie alias get`** function is for. You must run this function specifying the name of the alias (if you're not sure about the exact alias of the "genome", you can run `refgenie list` to get the aliases of all "genomes"):

```
(refgenie_v0.12) $ refgenie alias get -c genome_config.yaml -a plasmids
61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2
```

You can also get a list of the unique IDs of all the "genomes" that are stored within refgenie, by simply not indicating any specific aliases:

```
(refgenie_v0.12) $ refgenie alias get -c genome_config.yaml
                              Genome aliases                              
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┓
┃ genome                                           ┃ alias               ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━┩
│ 2230c535660fb4774114bfa966a62f823fdb6d21acf138d4 │ hg38                │
│ 94e0d21feb576e6af61cd2a798ad30682ef2428bb7eabbb4 │ rCRSd               │
│ 43fc48a366b42c3b8236f5e8b9d0ebc7831b37f46a924151 │ adenoviridae        │
│ eba23e46067df65a963f2fc3feddc0c54eb0bfb843a40115 │ anelloviridae       │
│ a612c3660e516039b6791589beb591d176a96ba34e54126b │ arenaviridae        │
│ 25b059484e0ec116eefa16fadb4ee1b31fba98a6245c7db1 │ astroviridae        │
│ 8c8a6981859c6f904323355953126ecb51590e97953fbefb │ caliciviridae       │
│ 219301dc0f09b85975c6beca9e45339c6e9e12faa90d575a │ caudovirales        │
│ 39c44e8f98d207ad5371106e424e5b145c12b3dd7d00c96d │ circoviridae        │
│ 57f64b91fb1b89ada0dd2abe23b20962be40dd1cc22c3ce9 │ coronaviridae       │
│ 1bd6d307dc944831d19c3b73cf3ade7f28c478f881cf05b1 │ flaviviridae        │
│ 31e2427365ce6266b10af53b5050dd871e20ff92b43dd068 │ hepeviridae         │
│ 2082db1b075c1173ef91e1d05570809d82bf7ea7b5e03400 │ matonaviridae       │
│ 49927764058a1467c344f1205667bf26b361cd290b88205c │ nairoviridae        │
│ 39065a0055c2ec6e2813bab86cd5136205bb61b0cf30e382 │ orthoherpesviridae  │
│ 907b08c910f0d79501efa6968a7d0a317f6c334561abe606 │ orthomyxoviridae    │
│ d1d77835dbc4ed501fd37bd3e26da8279ba73d04b5198ddf │ papillomaviridae    │
│ 0ef7a7181e0482422e7aa29a210bc76386e6bd2e7fff0c2b │ paramyxoviridae     │
│ 821fbc9b5ba42c4d3a77599c48135fb8d99493653c397490 │ parvoviridae        │
│ 1e1f0fd47de357ca615549424dea4ad548112e11f2ab0f1e │ phenuiviridae       │
│ ab5b94c306c090a0bbc4fe099360bf25886f0acdc9e38c93 │ picornaviridae      │
│ ...                                              │ ...                 │
└──────────────────────────────────────────────────┴─────────────────────┘
```

#### 7. Get the absolute path of a certain resource

Sometimes, you may want to know the full path of a certain resource. This is the purpose of the **`refgenie seek`** function. It is used in a very simple way, following the `genome/asset:tag` structure that was mentioned before.

Let's see an example:

```
(refgenie_v0.12) $ refgenie seek plasmids/annotations:v20200312 -c genome_config.yaml 
/data/ucct/bi/references/refgenie/alias/plasmids/annotations/v20200312/.
```

As you can see in the previous case, the function provides the full path of the resource that was requested. The path leads to a folder stored within `alias`, since no keys were specified (and in this case the files have no keys associated, but some other "genomes" do have keys for their corresponding tags, depending on the usefulness of these keys for some scripts).

This function allows the user to search of a specific file if it has a key linked to it. This is done replacing the **`genome/asset:tag`** structure by **`genome/asset.key:tag`**. Let's check an example:

```
(refgenie_v0.12) $ refgenie seek polycipiviridae/fasta.fasta:NC_035456.1 -c genome_config.yaml 
/data/ucct/bi/references/refgenie/alias/polycipiviridae/fasta/NC_035456.1/polycipiviridae.fa
```

Now, by following the previous structure, we got the full path of a certain fasta file, since it has the `fasta` key associated to it in the YAML configuration file.

#### 8. Rename a certain tag

Even though you might use this function in a limited number of times, refgenie allows the user to rename already existing tags, by means of the `refgenie tag` function. For example, let's say you made a mistake while adding a tag into the refgenie system and now the name of the tag is incorrectly written in the YAML configuration file, and also in the tags' folders. We can use this function to correct this.

If, for instance, you used an incorrect date to name a tag (like `v20180501`) and want to fix it in `data`, `alias` and the YAML file, you can use this function for that purpose. You have to indicate the current name of the tag, by means of the `genome/asset:tag` structure, and the new name you want to establish, by indicating the `-t` option:

```
(refgenie_v0.12) $ refgenie tag plasmids/ddbb:v20180501 -t v20200203 -c genome_config.yaml 
Default tag for 'plasmids/ddbb' set to: v20200203
Renamed directory: /data/ucct/bi/references/refgenie/data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203
Created alias directories:
 - /data/ucct/bi/references/refgenie/alias/plasmids/ddbb/v20200203
```

Originally, the YAML file would look like this:

```
ddbb:
        tags:
          v20180501:
            asset_path: /data/ucct/bi/references/refgenie/data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/
            asset_digest: f7655b11142f9e7fdc980ff827328bca
            seek_keys:
              ddbb: .
        default_tag: v20180501
```

But, after running `refgenie tag`, it will look like this:

```
ddbb:
        tags:
          v20200203:
            asset_path: /data/ucct/bi/references/refgenie/data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/
            asset_digest: f7655b11142f9e7fdc980ff827328bca
            seek_keys:
              ddbb: .
        default_tag: v20200203
```

>[!WARNING]
>**This function changes the name of a tag in the YAML configuration file, and also the corresponding tag folders both in `data` and `alias`**. However, the `asset_path` field in the YAML file is not updated, and all the symbolic links that were initially in the `alias` folder are lost after running `refgenie tag`.
>
>Initially, the corresponding `alias` folder would look like this after running `tree`:
>```
>(refgenie_v0.12) $ tree alias/plasmids/ddbb
>alias/plasmids/ddbb
>└── v20180501
>    ├── plasmids_may_2018_unnamed_term.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta
>    ├── plasmids_may_2018_unnamed_term.fasta.1.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta.1.bt2
>    ├── plasmids_may_2018_unnamed_term.fasta.2.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta.2.bt2
>    ├── plasmids_may_2018_unnamed_term.fasta.3.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta.3.bt2
>    ├── plasmids_may_2018_unnamed_term.fasta.4.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta.4.bt2
>    ├── plasmids_may_2018_unnamed_term.fasta.length -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta.length
>    ├── plasmids_may_2018_unnamed_term.fasta.rev.1.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta.rev.1.bt2
>    ├── plasmids_may_2018_unnamed_term.fasta.rev.2.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.fasta.rev.2.bt2
>    └── plasmids_may_2018_unnamed_term.length -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20180501/plasmids_may_2018_unnamed_term.length
>```
>But, after running `refgenie tag`, all the symbolic links are lost:
>```
>(refgenie_v0.12) $ tree alias/plasmids/ddbb
>alias/plasmids/ddbb
>└── v20200203
>```
>Therefore, running `refgenie tag` by itself is not enough to fix the issue.

As indicated in the previous warning message, the `refgenie tag` command cannot be run alone in order to fix the problem of an incorrect tag name, since the `asset_path` field in the YAML configuration file will stay incorrect and, plus, all the symbolic links from the `alias` folder will have been lost.

To fix this, after running `refgenie tag`, we have to:

1. Remove (by `rm -rf`) the `/alias/genome/asset/tag` folder. In the previous example, we should remove the newly created `v20200203` folder, which contains nothing.
2. Run `refgenie add` in order to re-add the tag into the YAML configuration file (even if it has already been added with `refgenie tag`, but the `asset_path` field is still wrong). After this, the `asset_path` field will now be correct and all the symbolic links will now appear in alias:

```
(refgenie_v0.12) $ refgenie add plasmids/ddbb:v20200203 --path /data/ucct/bi/references/refgenie/data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/ -c genome_config.yaml 
'61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb:v20200203' exists. Do you want to overwrite? [y/N] y
Will remove existing to overwrite
Default tag for '61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb' set to: v20200203
Added asset: 61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb:v20200203 
Created alias directories:
 - /data/ucct/bi/references/refgenie/alias/plasmids/ddbb/v20200203

(refgenie_v0.12) $ tree alias/plasmids/ddbb
alias/plasmids/ddbb
└── v20200203
    ├── plasmids_may_2018_unnamed_term.fasta -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta
    ├── plasmids_may_2018_unnamed_term.fasta.1.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta.1.bt2
    ├── plasmids_may_2018_unnamed_term.fasta.2.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta.2.bt2
    ├── plasmids_may_2018_unnamed_term.fasta.3.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta.3.bt2
    ├── plasmids_may_2018_unnamed_term.fasta.4.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta.4.bt2
    ├── plasmids_may_2018_unnamed_term.fasta.length -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta.length
    ├── plasmids_may_2018_unnamed_term.fasta.rev.1.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta.rev.1.bt2
    ├── plasmids_may_2018_unnamed_term.fasta.rev.2.bt2 -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.fasta.rev.2.bt2
    └── plasmids_may_2018_unnamed_term.length -> ../../../../data/61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb/v20200203/plasmids_may_2018_unnamed_term.length

1 directory, 9 files
```

>[!NOTE]
>As indicated before, don't worry if you are asked `'61489e0fbcd56c8be4a8ee12908009dbefaa90d9e84f0da2/ddbb:v20200203' exists. Do you want to overwrite? [y/N]`. You can say yes, and the tag (whose `asset_path` is wrong) will be removed, just to be re-written, this time correctly. No other tags from any other genomes will be affected by this.

## Useful resources

We hope this guide helps you understand how to use refgenie properly! 😊

For more information about refgenie, please have a look at these **resources**:
 * **https://refgenie.databio.org/en/latest/**
 * **https://refgenie.org/refgenie/**
 * **https://github.com/refgenie/refgenie**
 * **https://refgenie.databio.org/en/latest/demo_videos/**
 * **https://refgenie.databio.org/en/latest/usage/**