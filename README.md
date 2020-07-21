# 4C mapping
A mapping pipeline that maps and filters 4C data.

1. Note::::: Run bwa index on your fasta file first. Fasta file must have 'chr' in it.


In a 4C experiment DNA fragments are ligated to your fragment of interest, which are amplified using an inverse PCR. These fragments need to mapped back to the reference genome. Repetitive fragments need to removed from the analysis. Note that non-covered fragment are also of interest to the analysis, since they signal no interaction at this genomic location.

The analysis pipeline consists of three steps that has the user has to run:

#### 1. Creating a fragment map

To create a fragment map for you enzyme combination of choice please run the generate_fragment_map.pl script. For a fragment map for DpnII and Csp6I of the human genome you would use the following command:

```
## on harris:/DATA/t.severson/alex_4c
perl generate_fragment_map.pl ~/resources/hg19_sed.fa GATC CATG gatc_catc_fragment_map/
```

The fragment map will be strored in the directory `gatc_catc_fragment_map/`

#### 2. Identifying the repetitve fragments

We would like to filter the fragment map for repetitive fragments, therefore we will map all the fragments back to genome we selected them from to test whether they are unique or not. For the fragment map we just created will should run the following command:

```
mkdir 49_repeat
perl getRepeats.pl gatc_catc_fragment_map/ GATC 49 ~/resources/hg19_sed.fa 49_repeat/

## note, this will store your data into a folder called '49' in your test_repeat folder. 
## This will break the next script. Copy or move the data from '49' to the parent test_repeat directory.
```

The results will be placed in the directory 49_repeat/. Note the 49, this is the length of the ligated fragment including the restriction site. Note that for every different sequencing length for you 4C experiment, you will need to create a new repeat map. So if you have a sequence length of 65, a primer of 20nt and a 4nt restriction site, your sequence length should be `65 - 20 + 4 = 49`. 

In the case of Alex's first experiment, the primer is not exactly next to the restriction enzyme. The number should be calculated including the sequence between the primer and the restriction site: 75 (the length of the read) -20 (the length of the primer) -9 (sequence length between primer and cutting site for the first primer in Alex's list) + 4 (the cutting site sequence). So for the first primer, the sequence length should be `75 - 20 - 9 + 4 = 50`

#### 3. Splitting FASTQ and mapping to the genome

The preprocessing of the data is now finished and you can start to map your data to the genome. The only thing you need is an index file, which contains the minimal information of your 4C experiment. Add the 1st restriction enzyme onto the end of your primer sequence. The structure of this file is as follows:

|Experiment name | primer sequence | path to reference genome | restriction enzyme 1 | restriction enzyme 2 | viewpoint chromosome |
|---------- | ---------- | ----------|----------|----------|----------|
|Nanog_enhancer | CACATTGATTAACCTA**GATC** | /data/reference/mm9.fa | GATC | GTAC | chr6 |

Note that the reference should also have bwa index. Also note that the second restriction enzyme is not strictly necessary, but the chromosome id should always be in the 6th column. Given the curre
nt setup it is not possible to mix restriction enzyme combination or reference genomes. If you have multiple genomes or multiple restriction enzyme combinations please create a seperate index file
for each one.

The following command is used to process and map your data.

```
perl mapping_pipeline.pl simple_index.txt test_run 4C_data.fastq.gz 10 gatc_catc_fragment_map/ 50_repeat/
```

More detailed information is given in the scripts themselves.

###Requirements:

To run the 4C mapping you will need the following software installed on your machine

1. bwa (bwasw)
2. bedtools (fastaFromBed)
3. Inline::C

