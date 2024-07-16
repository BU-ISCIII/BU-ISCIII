# Handle sample resequencing

Sometimes, a sequencing run has to be repeated. Some of the possible causes are:

- The samples do not achieve the desired depth of the target organism (expired libraries or low-performance libraries are used because they are available and there is a hurry)
- The sequencing has poor quality, with few PF Clusters (Passing Filter clusters)
- R2 has failed and can only be used as single-end
- Others

In these cases, the researcher decides to resequence the samples in another run, and they usually provide the same name in the sequencing samplesheet. Therefore, upon demultiplexing, the samples might be named exactly the same, causing some issues when creating symbolic links in the RAW folder.

In these cases, there are two ways to proceed:

1. If the samples are named exactly the same in `/srv/fastq_repo` (e.g., `/srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S2_R1_001.fastq.gz` and `/srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S2_R1_001.fastq.gz`).
     - In this case, we cannot use the bu-isciii tools. The symbolic links must be created manually.
     - When creating the symbolic link in RAW, we will rename the samples from the second run, adding the name of the second run, so that the RAW folder looks something like this:

     ```bash
     20130635_S293_R1_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S293_R1_001.fastq.gz
     20130635_S293_R2_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S293_R2_001.fastq.gz
     20150424_S296_R1_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20150424_S296_R1_001.fastq.gz
     20150424_S296_R2_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20150424_S296_R2_001.fastq.gz
     20130635-NovaSeq102_R1.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S293_R1_001.fastq.gz
     20130635-NovaSeq102_R2.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S293_R2_001.fastq.gz
     20150424-NovaSeq102_R1.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20150424_S296_R1_001.fastq.gz
     20150424-NovaSeq102_R2.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20150424_S296_R2_001.fastq.gz
     ```

2. If the samples are not named exactly the same in `/srv/fastq_repo` (because they differ in the _S) (e.g. `/srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S2_R1_001.fastq.gz` and `/srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S25_R1_001.fastq.gz`).
    - In this case, it can be done with the bu-isciii tools.
    - Once we have used the bu-isciii tools, the files will need to be renamed manually to match exactly as in the previous case:

    ```bash
    20130635_S293_R1_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S293_R1_001.fastq.gz
    20130635_S293_R2_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S293_R2_001.fastq.gz
    20150424_S296_R1_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20150424_S296_R1_001.fastq.gz
    20150424_S296_R2_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20150424_S296_R2_001.fastq.gz
    20130635-NovaSeq102_R1.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S293_R1_001.fastq.gz
    20130635-NovaSeq102_R2.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S293_R2_001.fastq.gz
    20150424-NovaSeq102_R1.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20150424_S296_R1_001.fastq.gz
    20150424-NovaSeq102_R2.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20150424_S296_R2_001.fastq.gz
    ```

In these cases, what we are going to do is create different `samples_id.txt` files. We will have:

- `samples_id.txt`: This will contain the list of ALL the samples analyzed. In this specific example, it would look like this:

```bash
20130635
20150424
20130635-NovaSeq102
20150424-NovaSeq102
```

- `samples_id01.txt`: This will contain only the samples from the first delivery01. It would look like this:

```bash
20130635
20150424
```

- `samples_id02.txt`: This will contain only the samples from the second delivery02. It would look like this:

```bash
20130635-NovaSeq102
20150424-NovaSeq102
```

Next, what we will do is create a symbolic link in `00-reads` for the new samples, so that the 00-reads folder will look like this:

```bash
20130635_R1.fastq.gz
20130635_R2.fastq.gz
20150424_R1.fastq.gz
20150424_R2.fastq.gz
20130635-NovaSeq102_R1.fastq.gz
20130635-NovaSeq102_R2.fastq.gz
20150424-NovaSeq102_R1.fastq.gz
20150424-NovaSeq102_R2.fastq.gz
```

> [!IMPORTANT]  
> This is slightly different in the case you have to **concatenate the sample's reads**.

When we have the scenario where the **samples are going to be concatenated**, for example, to increase sequencing depth, it is important to ensure that the reads can be concatenated, which means they must have been processed in the same way or even come from the same library. The first steps in the RAW folder are the same as in the previous case; we will create symbolic links to the samples in `/srv/fastq_repo`, renaming those from the second run. However, we will need to add a `lablog` where we have, for example in our case:

```bash
cat 20130635_S293_R1_001.fastq.gz 20130635-NovaSeq102_R1.fastq.gz > 20130635-merged_R1.fastq.gz
cat 20130635_S293_R2_001.fastq.gz 20130635-NovaSeq102_R2.fastq.gz > 20130635-merged_R2.fastq.gz
cat 20150424_S296_R1_001.fastq.gz 20150424-NovaSeq102_R1.fastq.gz > 20150424-merged_R1.fastq.gz
cat 20150424_S296_R2_001.fastq.gz 20150424-NovaSeq102_R2.fastq.gz > 20150424-merged_R2.fastq.gz
```

So that the RAW folder would look like this:

```bash
20130635_S293_R1_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S293_R1_001.fastq.gz
20130635_S293_R2_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20130635_S293_R2_001.fastq.gz
20150424_S296_R1_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20150424_S296_R1_001.fastq.gz
20150424_S296_R2_001.fastq.gz -> /srv/fastq_repo/MiSeq_GEN_235_20240702_user/20150424_S296_R2_001.fastq.gz
20130635-NovaSeq102_R1.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S293_R1_001.fastq.gz
20130635-NovaSeq102_R2.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20130635_S293_R2_001.fastq.gz
20150424-NovaSeq102_R1.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20150424_S296_R1_001.fastq.gz
20150424-NovaSeq102_R2.fastq.gz -> /srv/fastq_repo/NovaSeq_GEN_102_202408015_user/20150424_S296_R2_001.fastq.gz
20130635-merged_R1.fastq.gz
20130635-merged_R2.fastq.gz
20150424-merged_R1.fastq.gz
20150424-merged_R2.fastq.gz
```

The next step is to create the `samples_id`, so that, just like in the previous case, we will still have several `samples_ids` but in this case, instead of adding `sample-NovaSeq102`, we will add `sample-merged`. They would look like this:

- `samples_id.txt`: This will contain the list of ALL the samples analyzed. In this specific example, it would look like this:

```bash
20130635
20150424
20130635-merged
20150424-merged
```

- `samples_id01.txt`: This will contain only the samples from the first delivery01. It would look like this:

```bash
20130635
20150424
```

- `samples_id02.txt`: This will contain only the samples from the second delivery02. It would look like this:

```bash
20130635-merged
20150424-merged
```

Next, what we will do is create a symbolic link in `00-reads` for the new samples, so that the 00-reads folder will look like this:

```bash
20130635_R1.fastq.gz
20130635_R2.fastq.gz
20150424_R1.fastq.gz
20150424_R2.fastq.gz
20130635-merged_R1.fastq.gz
20130635-merged_R2.fastq.gz
20150424-merged_R1.fastq.gz
20150424-merged_R2.fastq.gz
```

> [!IMPORTANT]
> Don't forget to change the `samples_id.txt` to `samples_id2.txt` in all the lablogs of the new analyses so that the correct samples are analyzed.