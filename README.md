# Stentor pyriformis gene prediction
#### By: Ashley Albright

This repository contains a description of annotating an assembled genome for Stentor pyriformis with all accompanying code. 

Preprint: TBD

Authors: TBD

Final files deposited here: TBD


### 1. Align RNA-seq reads to the assembled pyriformis genome

Similar to [Singh et al 2022](https://www.biorxiv.org/content/10.1101/2021.12.14.471607v4.full), we modified the source code of HISAT2 to accomodate short introns. In hisat2.cpp, minIntronLen was lowered to 9 from 20. 

- Build hisat2 index

  -  -p 4 specifies the number of threads to use

```
/Users/aralbright/hisat2/hisat2-build -p 4 /Volumes/albright_postdoc/pyriformis/stentor_pyriformis.20210302_final.fasta
pyriformis_index
```

- Run hisat2

  Parameters:
  - --min-intron-len = 9
  - --max-intron-len = 101
  - --very-sensitive

Min and max intron length were decided based on prior work describing the short introns in a closely related species, Stentor coeruleus.

```
/Users/aralbright/hisat2/hisat2 -x /Volumes/albright_postdoc/pyriformis/genome/genome -p 4 --min-intronlen 9 --max-intronlen 101 --very-sensitive -1 /Volumes/albright_postdoc/pyriformis/genome/STENTOR_RNA_GGAGCGTC_GTCCGTGC_S1_L001_R1_001.fastq.gz -2 /Volumes/albright_postdoc/pyriformis/genome/STENTOR_RNA_GGAGCGTC_GTCCGTGC_S1_L001_R2_001.fastq.gz > sample.sam
```

Output:

```
17130914 reads; of these:
  17130914 (100.00%) were paired; of these:
    6598631 (38.52%) aligned concordantly 0 times
    10027693 (58.54%) aligned concordantly exactly 1 time
    504590 (2.95%) aligned concordantly >1 times
    ----
    6598631 pairs aligned concordantly 0 times; of these:
      3922613 (59.45%) aligned discordantly 1 time
    ----
    2676018 pairs aligned 0 times concordantly or discordantly; of these:
      5352036 mates make up the pairs; of these:
        155153 (2.90%) aligned 0 times
        4172309 (77.96%) aligned exactly 1 time
        1024574 (19.14%) aligned >1 times
99.55% overall alignment rate
```

#### .sam to .bam, indexing
```
samtools view -S -b sample.sam > sample.bam
samtools sort sample.bam -o sample.sorted.bam
samtools index sample.sorted.bam
```

### 2. Run Intronarrator/Augustus


Source: https://github.com/Swart-lab/Intronarrator

Paper: https://www.biorxiv.org/content/10.1101/2021.12.14.471607v4.full

Intronarrator is a wrapper for Augustus developed in order to accomodate for short introns. The program predicts and removes introns prior to running Augustus using an intron-less model, then adds the introns back in. I'd like to acknowledge Estienne Swart here for helping me getting this to run.

I followed their instructions for running Intronarrator (in the github linked above), with a few exceptions because I ran this on an Apple computer. I installed Augustus with Homebrew and added a species folder called 'StentorV2' containing the files necessary to predict genes. These files originated from work with the Stentor coeruleus genome.

I changed the following parameters in my version of intronarrator.sh:

- INTRONARRATOR_PATH=/Users/aralbright/Intronarrator
- MAX_INTRON_LEN=101
- GENETIC_CODE=1

Stentor coeruleus uses a standard genetic code (1), and I made that assumption here with pyriformis.

```
./intronarrator.sh /usr/local/Cellar/augustus/3.5.0_1 /usr/local/Cellar/augustus/3.5.0_1/config StentorV2 stentor_pyriformis.20210302_final sample.sorted
```

### 3. Final checks

I double checked the predictions by making sure the length of coding sequences were divisible by 3.

```
cat stentor_pyriformis.20210302_final.0.2.final.CDS.fa | awk '$0 ~ ">" {if (NR > 1) {print c;} c=0;printf substr($0,2,100) "\t"; } $0 !~ ">" {c+=length($0)/3;} END { print c; }' > CDS_count.txt
```

```
awk '{if  ( $2 ~ /\./ ) { print $} }' CDS_count.txt
```

I found one transcript that is not divisible by 3, implying that there was an error we need to track down. For future reference: contig_203.0-203133.g959.t1

I also double checked the annotations against the RNA-seq in IGV. 

### 4. GFF Parser + Contig/Gene Name Changer 

To standardize the contig and gene names, I modified a Drosophila melanogaster GFF parser written by Colleen Hannon and Mike Eisen to fit the format of my GFF file. See change_gff_ids.ipynb in this folder for the code. 

See change_fasta_contigs.ipynb for changing the contig names in a fasta file. 






