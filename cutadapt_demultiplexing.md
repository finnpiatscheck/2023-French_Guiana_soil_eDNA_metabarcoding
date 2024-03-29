# Demultiplexing with (stand-alone) Cutadapt

The following example is for the librarie AMVG06 (marker 18S Eukaryote for savanas and pastures soils, project BING).

### Install Cutadapt

If miniconda is installed, simply run:

```
conda install -c bioconda cutadapt
```
https://anaconda.org/bioconda/cutadapt

If desired, the program can be installed in its own environment.
The version of cutadapt used in this analyses is cutadapt v4.4.

### Difficulties running cutadapt with large datasets

When using Cutadapt with a large number of samples to demultiplex, I often get the error ```Error: [Errno 24] Too many files open```. From the discussion https://github.com/marcelm/cutadapt/issues/320 I tried to raise the soft limit with:
```
ulimit -S -n 10240
```
Then, the following commmands worked.

### Demultiplex mixed-orientation paired-end reads with Cutadapt

For reference, read: https://cutadapt.readthedocs.io/en/stable/guide.html?highlight=orientation#demultiplexing-paired-end-reads-in-mixed-orientation.

When dealing with mixed orientation reads, we demultiplex twice. 

We can use only tags (barcodes) to demultiplex. However this method will result in false assignments because f and r tags are identical (e.g., r1 and f1 are the same sequences). For example, in the forward orientation, I will demultiplex reads taged f1 (R1) r2 (R2) and r1 (R1) f2 (R2) and assign them wrongly to the same sample. For this reason, Cutadapt needs to concider the primers. The schemes of demultiplexing will look like this:

    |--------|----------------------|----------------------------> R1
       TAG            PRIMER               BIOLOGICAL SEQUENCE

                                        R2 <----------------------------|----------------------|--------|
                                                 BIOLOGICAL SEQUENCE             PRIMER            TAG    

The forward and reverse "barcode" information will be the tag and the primer concatenated together.

However, as stated aboven, reads are in mixed orientation, which means that R1 reads and their pair can be found in the R2.fastq file and vice versa. For this reason, in a first round of demultiplexing, the reversed reads pairs will be unasigned. So, demultiplexing will occur twice, by concatenating all the unasigned and wrongly assigned reads (reads assigned to unused tag combination) and demultiplex again by simply reversing the R1 and R2 files.

Thus, we organize the demultiplexing folder as follow:
```
├───round1
│   ├───assigned
│   ├───unknown_to_concatenate
│   └───unused_combinaisons
├───round2
│   ├───assigned
│   ├───unassigned
│   └───unused_combinaisons
└───assigned_reads

```
The adapter FASTA files are placed in the main folder and the sequence FASTQ files are placed in the ```round1``` folder. Adapter file format can be found in the cutadapt documentation. In short, the ID is the tag, and the sequence following is the tag+primer sequence.

### Parameter settings

To visualize if the reads were sorted correctly by checking the assigned and unassigned reads with the command ```zless```, we add the flag ```--action retain``` that will allow to retain the adapters.

Because we include the primers and tags, Cutadapt will attempt to align the input adapters to the reads and allow some mismatches due to seuqencing errors. First, we will allow the value of 10% mismatches (which, for example, an adapter length of 26bp (tag+primer) represents 2 nucleotides mismatche). Thus, we use the flag ```-e 0.10```. Next we will allow an overlap shorter than the length of the adapter. This is because sometimes, reads start after the first nucleotide of the tag. We use the flag ```-O 23``` (3 nucleotides shorter than 26bp primer+tag length). The value will change depending on the length of the longest adapter used for the librairie. 

Finally, we run this computation on 12 cores, we add the flag ```-j 12```. This value can be changed as desired.


### Round 1
Raw reads are placed in main folder. The following command should be run from the round 1 folder.
```
cutadapt -g file:../AMVG06_cutadapt_barcodes_primers_forward.txt \
    -G file:../AMVG06_cutadapt_barcodes_primers_reverse.txt \
    -j 12 \
    -e 0.10 \
    -O 23 \
    --action retain \
    -o round1-{name1}-{name2}.R1.fastq.gz \
    -p round1-{name1}-{name2}.R2.fastq.gz \
    ../AMVG06_R1.fastq.gz ../AMVG06_R2.fastq.gz
```
The resulting files are named with the prefix "round1" followed by the tag combinaison. Files with "unknown" had one or both tag unidentified.


### Round 2

Next we concatenate all the unassigned or wrongly assigned reads. But first we need to move all assigned reads in a folder 
```assigned``` and unassigned (unknown for now) in another folder ```unknown_to_concatenate```. All files with "unknown" or with adapter combinations unused (for example if f24 was not used, reads assigned to this tag should be reconsidered). 

#### Move the unused tag combinaisons (due to tag jumping) into the ```unused_combinaisons``` folder.
```
mv round1-TACTATAC-GTCACGTC* unused_combinaisons
mv round1-GTGTACAT-GACTGATG* unused_combinaisons
mv round1-TACTATAC-GTACGACT* unused_combinaisons
mv round1-ACTAGATC-ACGACGAG* unused_combinaisons
mv round1-GATCGCGA-ACGACGAG* unused_combinaisons
mv round1-TACTATAC-CTGCGTAC* unused_combinaisons
...

```

#### Move assigned reads from rounds 1
```
mv round1-ACACACAC-ACACACAC* ./assigned
mv round1-ACAGCACA-ACACACAC* ./assigned
mv round1-GTGTACAT-ACACACAC* ./assigned
mv round1-TATGTCAG-ACACACAC* ./assigned
mv round1-TAGTCGCA-ACACACAC* ./assigned
mv round1-TACTATAC-ACACACAC* ./assigned
...
```

#### Move all the other files into the unknown folder (only flles with "unkown" should remain)
```
mv *unknown*.gz ./unknown_to_concatenate 
```

Then, in the ```unknown_to_concatenate``` folder we concatenate all these file.

```
cat *R1* > unknown.R1.fastq.gz
cat *R2* > unknown.R2.fastq.gz
```
We move the two unknown files to round 2.
```
mv unknown.R?.fastq.gz ../../round2
```



In the folder named round2, we rename unknown.R1.fastq.gz into unknown.R2.fastq.gz and unknown.R2.fastq.gz into unknown.R1.fastq.gz.

```
mv unknown.R1.fastq.gz unknown.R1.R2.fastq.gz
mv unknown.R2.fastq.gz unknown.R1.fastq.gz
mv unknown.R1.R2.fastq.gz unknown.R2.fastq.gz
```

Then we run Cutadapt again for round 2:

```
cutadapt -g file:../AMVG06_cutadapt_barcodes_primers_forward.txt \
    -G file:../AMVG06_cutadapt_barcodes_primers_reverse.txt \
    -j 12 \
    -e 0.10 \
    -O 23 \
    --action retain \
    -o round2-{name1}-{name2}.R1.fastq.gz \
    -p round2-{name1}-{name2}.R2.fastq.gz \
    unknown.R1.fastq.gz unknown.R2.fastq.gz
```
Once done, we move assigned reads to their corresponding folder.
```
mv round2-ACACACAC-ACACACAC* ./assigned
mv round2-ACAGCACA-ACACACAC* ./assigned
mv round2-GTGTACAT-ACACACAC* ./assigned
mv round2-TATGTCAG-ACACACAC* ./assigned
mv round2-TAGTCGCA-ACACACAC* ./assigned
mv round2-TACTATAC-ACACACAC* ./assigned
...
```
Move unassigned reads and unused combinaisons to their corresponding folders.
```
mv round2*unknown* unassigned
mv round2* unused_combinaison/
```
Then in both round 1 and round 2 assigned folders and unused_combinaisons, we copy (or move for space) all the assigned reads to the corresponding folder. The unused_combinaisons should be assigned to a g
```
mv round2-* ../../assigned_reads/
mv round1-* ../../assigned_reads/
```

### Rename correctly assigned reads by sample name and concatenate round 1 and round 2

Now all the reads should be properly sorted, but the files are not named properly. 
We use the program ```rename``` for this:

All the round 1 and round 2 assigned reads should be moved together in a new folder ```assigned_reads``` and reads renamed and concatenated by samples.
```
mv round1/assigned/round1* assigned_reads
mv round2/assigned/round2* assigned_reads
```
Then we rename the files correctly:
```
rename 's/ACACACAC-ACACACAC.R/RB.1.R/' *.fastq.gz
rename 's/ACAGCACA-ACACACAC.R/RB.2.R/' *.fastq.gz
rename 's/GTGTACAT-ACACACAC.R/RB.3.R/' *.fastq.gz
rename 's/TATGTCAG-ACACACAC.R/RB.4.R/' *.fastq.gz
rename 's/TAGTCGCA-ACACACAC.R/RB.5.R/' *.fastq.gz
rename 's/TACTATAC-ACACACAC.R/SAM.1.R/' *.fastq.gz
...
```

The samples not renamed are used tag combinaisons. But they should not be any if unused tag combinaisons were separated from these files previously.

Next we concatenate round 1 and round 2 files.

```
cat round?-RB.1.R1.fastq.gz > RB.1.R1.fastq.gz
cat round?-RB.2.R1.fastq.gz > RB.2.R1.fastq.gz
cat round?-RB.3.R1.fastq.gz > RB.3.R1.fastq.gz
cat round?-RB.4.R1.fastq.gz > RB.4.R1.fastq.gz
cat round?-RB.5.R1.fastq.gz > RB.5.R1.fastq.gz
cat round?-SAM.1.R1.fastq.gz > SAM.1.R1.fastq.gz
...

cat round?-RB.1.R2.fastq.gz > RB.1.R2.fastq.gz
cat round?-RB.2.R2.fastq.gz > RB.2.R2.fastq.gz
cat round?-RB.3.R2.fastq.gz > RB.3.R2.fastq.gz
cat round?-RB.4.R2.fastq.gz > RB.4.R2.fastq.gz
cat round?-RB.5.R2.fastq.gz > RB.5.R2.fastq.gz
cat round?-SAM.1.R2.fastq.gz > SAM.1.R2.fastq.gz
...
```

Remove intermediate files:
```
rm round*
```

#### The number of files should match the number of samples
```
ls | wc -l
```

Demultiplexing reads per sample is done.

### Calculating the statistics on how much reads are lost in the process

#### Count number of sequences in raw data and demultiplexed

```
zcat AMVG06_R* | wc -l
cd round1/assigned
zcat round1* | wc -l
cd round1/unknown_to_concatenate
zcat round1* | wc -l
cd round1/unused_combinaisons
zcat round1* | wc -l
cd round2/assigned
zcat round2* | wc -l
cd round2/unknown_to_concatenate
zcat round2* | wc -l
cd round2/unused_combinaisons
zcat round2* | wc -l
```
The values are the number of lines in R1 and R2 files. For the actual number of reads, this value needs to be divided by 8.


### Demultiplexing DIAMOND and GALBAO data

The process is pretty much the same, except that 40 forward barcode and 36 reverse barcode are being used to identify 1440 tag combinaisons.


## Trim the tags (optional)

In the previous demultiplexing with cutadapt, tags and primers were not removed on purpose. This has no repercution when using APSCALE next, even if APSCALE expects reads with trimmed tags but primers still present. APSCALE also uses cutadapt and will search for the primers and trim all previous nucleotides, including the tags when it founds them. 
```
mkdir trim_barcodes
cp assigned_reads/*.gz trim_barcodes
cd trim_barcodes
mkdir trimmed
```
All tags and primers should be anchored at the beginning of the reads since they have been already demultiplexed with adapters retained and the few previous nucleotides were removed.
```
for file in *.fastq.gz; do 
    cutadapt -g ^file:../DIAMOND_barcodes.fasta \
    -j 12 \
    -e 0.10 \
    -O 3 \
    --action trim \
    -o trimmed/$file \
    $file  
done 
```
Warnings will appear that some barcodes are within 1 error of each other. In some case, the wrong tag will be trimmed at the beginning of the read. But in this case, it has no repercution on the analysis because we want all the tags gone.

