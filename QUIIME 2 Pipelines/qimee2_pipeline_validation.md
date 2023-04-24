# QIIME2 Pipeline validation

This script is an attempt to work through the qiime2 pipeline entirely, understanding each steps, with a subset of data to make the computationally intensive tasks faster and validating the methods used before working the full datasets. Here, there won't be detailed explanation of each steps, but rather exploring and discussing the outputs.
<br />

---

The stop sign &#x1F6D1; will indicate my progress by showing the step that I am at.

---


## Subsample with seqtk

I chose to subset the 16S Bacteria dataset, originally with 7,512,428 reads in two file of ~2.7Gb.

### Install seqtk
https://github.com/lh3/seqtk

```
cd ~/work/programs/
git clone https://github.com/lh3/seqtk.git;
cd seqtk
make
```

### Run seqtk to subsample 100,000 paired-end reads
I will subset 100,000 paired-end reads in both R1 and R2 files.

Change the directory to the one containing the data and run the following code:

```
 ~/work/programs/seqtk/seqtk sample -s150 220819_SN1126_A_L001_AMVG-26_R1.fastq.gz 100000 > 220819_SN1126_A_L001_AMVG-26_R1_subset_100K.fastq.gz

 ~/work/programs/seqtk/seqtk sample -s150 220819_SN1126_A_L001_AMVG-26_R2.fastq.gz 100000 > 220819_SN1126_A_L001_AMVG-26_R2_subset_100K.fastq.gz
```
*NOTES: Remember to set the same seed for each FASTQ file to keep the pairing*  

The resulting R1 and R2 FASTQ files are ~65Mb each. I move the files in their own directory for further test the pipeline with this subset.
```
mkdir subsamples
mv 220819_SN1126_A_L001_AMVG-26_R1_subset_100K.fastq.gz 220819_SN1126_A_L001_AMVG-26_R2_subset_100K.fastq.gz subsamples/
```



<br />

---

## Import the data, demultiplex and summarize the data

First we create a manifest.tsv file that will contain the path to the data and the name of the FASTQ files to be analyzed.

```
echo -e 'sample-id\tforward-absolute-filepath\treverse-absolute-filepath\n16_bacteria\t/home/ECOFOG/finn.piatscheck/work/data/16S_bacteria/subsamples/220819_SN1126_A_L001_AMVG-26_R1_subset_100K.fastq.gz\t/home/ECOFOG/finn.piatscheck/work/data/16S_bacteria/subsamples/220819_SN1126_A_L001_AMVG-26_R2_subset_100K.fastq.gz' > manifest.tsv
```


```
conda activate qiime2-2023.2
```

### Import the data
```
qiime tools import \
    --type 'SampleData[PairedEndSequencesWithQuality]' \
    --input-path ~/work/analysis/qiime2-pipeline_validation/manifest.tsv \
    --input-format PairedEndFastqManifestPhred33V2 \
    --output-path 16S_subset_100K_mux-paired-end.qza
```
The above works for demultiplex data, most of my samples are still multiplexed in the raw sequence dataset so for multiplexed data I shall try the following.

```
qiime tools import \
    --type 'MultiplexedPairedEndBarcodeInSequence' \
    --input-path ~/work/data/16S_bacteria/subsamples/renamed_qiime/ \
    --output-path 16S_subset_100K_mux-paired-end.qza
```
*NOTES: the files need to be called ```forward.fastq.gz``` and ```reverse.fastq.gz```, and be compressed. My files were .gz but uncompressed somehow, I had to compresse them with ```gzip -r```.*

### Demultiplex with cutadapt

```
qiime cutadapt demux-paired \
    --i-seqs 16S_subset_100K_mux-paired-end.qza \
    --m-forward-barcodes-file amvg26_metadata.tsv \
    --m-forward-barcodes-column barcode_forward \
    --m-reverse-barcodes-file amvg26_metadata.tsv \
    --m-reverse-barcodes-column barcode_reverse \
    --o-per-sample-sequences 16S_subset_100K_per-sample_sequence.qza \
    --o-untrimmed-sequences   16S_subset_100K_untrimmed-sequences.qza  
```
  
<br />

### Summarize reads information

```
qiime demux summarize \
  --i-data 16S_subset_100K_per-sample_sequence.qza \
  --o-visualization 16S_subset_100K_per-sample_sequence.qzv
```

## Estimate truncation parameters with FIGARO

I ran Figaro with the dedicated script. See the figaro.md pipeline to get truncation parameters.

Here was the output working with this subset of reads.

```
{
        "trimPosition": [
            288,
            64
        ],
        "maxExpectedError": [
            2,
            1
        ],
        "readRetentionPercent": 89.78,
        "score": 88.781
    }
```
In qiime2, we will truncate reads at 288bp for the F reads and 64bp for the R reads.

*NOTES: 64bp seems really short. Am I sure this is the right parameter values to use? Maybe, the amplicon size is 295bp so with 288bp and 64bp I should have enough overlap to capture the biological sequence.*


## Denoise with DADA2 and build feature table

```
  qiime dada2 denoise-paired \
    --i-demultiplexed-seqs 16S_subset_100K_per-sample_sequence.qza \
    --p-trim-left-f 28 \
    --p-trim-left-r 25 \
    --p-trunc-len-f 288 \
    --p-trunc-len-r 64 \
    --p-chimera-method consensus \
    --p-n-threads 16 \
    --o-representative-sequences 16S_subset_100K_representative-sequences.qza \
    --o-table 16S_subset_100K_table.qza \
    --o-denoising-stats 16S_subset_100K_denoising-stats.qza
```

```
qiime metadata tabulate \
  --m-input-file 16S_subset_100K_denoising-stats.qza \
  --o-visualization 16S_subset_100K_denoising-stats.qzv
```

### Summarize

```
qiime feature-table summarize \
  --i-table 16S_subset_100K_table.qza \
  --m-sample-metadata-file amvg26_metadata.tsv\
  --o-visualization 16S_subset_100K_table.qzv
```

```
qiime feature-table tabulate-seqs \
  --i-data 16S_subset_100K_representative-sequences.qza \
  --o-visualization 16S_subset_100K_representative-sequences.qzv
```


## Taxonomic assignment

### Download the SILVA classifier

```
wget \
-O "silva-138-99-nb-classifier.qza" \
https://data.qiime2.org/2023.2/common/silva-138-99-nb-classifier.qza
```


### Classification
```
qiime feature-classifier classify-sklearn \
   --i-classifier silva-138-99-nb-classifier.qza \
   --i-reads 16S_subset_100K_representative-sequences.qza \ 
   --o-classification 16S_subset_100K_taxonomy_SKLEARN.qza
```

### Visualization
```
qiime metadata tabulate \
  --m-input-file 16S_subset_100K_taxonomy_SKLEARN.qza \
  --o-visualization 16S_subset_100K_taxonomy_SKLEARN.qzv
```
```
qiime metadata tabulate \
  --m-input-file 16S_subset_100K_taxonomy_SKLEARN.qza  \
  --m-input-file 16S_subset_100K_representative-sequences.qza  \
  --o-visualization 16S_subset_100K_annotated_rep-set.qzv
```
```
qiime taxa barplot \
  --i-table 16S_subset_100K_table.qza \
  --i-taxonomy 16S_subset_100K_taxonomy_SKLEARN.qza \
  --m-metadata-file amvg26_metadata.tsv \
  --o-visualization 16S_taxa-bar-plots_SKLEARN.qzv
```


##  Phylogenetic tree


```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences 16S_subset_100K_representative-sequences.qza \
  --o-alignment 16S_subset_100K_aligned-rep-seqs.qza \
  --o-masked-alignment 16S_subset_100K_masked-aligned-rep-seq.qza \
  --o-tree 16S_subset_100K_unrooted-tree.qza \
  --o-rooted-tree 16S_subset_100K_rooted-tree.qza
```
<br />


&#x1F6D1; 


## Analysis of Alpha and Beta diversities

```
qiime diversity alpha-rarefaction \
  --i-table 16S_subset_100K_table.qza \
  --i-phylogeny 16S_subset_100K_rooted-tree.qza \
  --p-max-depth 8000 \
  --m-metadata-file amvg26_metadata.tsv \
  --o-visualization 16S_subset_100K_alpha-rarefaction.qzv
```

```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny 16S_rooted-tree.qza \
  --i-table 16S_table.qza \
  --p-sampling-depth 1000 \
  --p-n-jobs-or-threads 16 \
  --m-metadata-file 16S_sample-metadata.tsv \
  --output-dir 16S_core-metrics-results
```
