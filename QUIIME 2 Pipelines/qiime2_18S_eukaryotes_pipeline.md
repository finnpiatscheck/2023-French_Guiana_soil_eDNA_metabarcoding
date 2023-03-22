# QIIME2 Pipeline for 18S Metabarcoding Analysis

---

In this pipeline I will use QIIME2 to perform an analysis of 18S reads of soil samples from savanas, pastures, and hurban soils in French Guiana, in order to identify and analyse soil eukaryotes communities. The following pipeline on 16S data is adapted from the physalia course on metbarcoding for microbial ecology (course 30).

---

In this pipeline, it is assumed that QIIME2 was installed correctly. See the QIIME2 installation  pipeline for installation instructions.

---

The stop sign &#x1F6D1; will indicate my progress by showing the step that I am at.

---

# 18S Eukaryotes 

---

&#x1F6D1; This script is a copy of the the 16S pipeline, which I haven't worked on yet.

---

```
mkdir 18S-qiime2
```
```
cd 18S-qiime2
```
### Activate the conda environment
```
conda activate qiime2-2023.2
```
<br />

---

## STEP1: Importing data, summarize the results, and examining quality of the reads
<br />
The data type is a FastQ file, which correspond to: 
```SampleData[PairedEndSequencesWithQuality]```.

The format list is quite large. Our files are somewhat like Casava 1.8 paired-end demultiplexed data. We would indicate: 
```CasavaOneEightSingleLanePerSampleDirFmt```.
However the format is slightly different. We use ```PairedEndFastqManifestPhred33V2```

```
qiime tools import \
    --type 'SampleData[PairedEndSequencesWithQuality]' \
    --input-path ~/work/data/18S_bacteria/manifest.txt \
    --input-format PairedEndFastqManifestPhred33V2 \
    --output-path 18S_demux-paired-end.qza
```

---

## STEP2: Quality controlling sequences and building Feature Table and Feature Data
<br />
Get some statistics about the raw reads. \

```
qiime demux summarize \
  --i-data 18S_demux-paired-end.qza \
  --o-visualization 18S_demux-paired-end.qzv
```
The visualization of the quality plot of the reads (with QUIIME2view or performed with FASTQC) shows that forward read quality drops below 30 after ~270bp, and that reverse read quality drops below 30 after ~160bp. We will trunk the sequences with these values in the following denoising step. However, it can be considered like a harsh truncation. I will reconsider this step with superior sequence conservation. \
<br />
OPTIONAL: Use FIGARO for optimizing trimming parameters.
https://www.biorxiv.org/content/10.1101/610394v1 \
http://john-quensen.com/tutorials/figaro/  
See the FIGARO pipeline for more information.
<br />


### Denoising with DADA2:
The code below takes quite some computational resource. Several chimera removal methods can be chosen.

```
  qiime dada2 denoise-paired \
    --i-demultiplexed-seqs 18S_demux-paired-end.qza \
    --p-trim-left-f 13 \
    --p-trim-left-r 13 \
    --p-trunc-len-f 270 \
    --p-trunc-len-r 160 \
    --p-chimera-method consensus \
    --p-n-threads 16 \
    --o-representative-sequences 18S_representative-sequences.qza \
    --o-table 18S_table.qza \
    --o-denoising-stats 18S_denoising-stats.qza
```

Next we generate a qzv file containing the table summarizing the denoising process to be visualized.
```
qiime metadata tabulate \
  --m-input-file 18S_denoising-stats.qza \
  --o-visualization 18S_denoising-stats.qzv
```
<br />

---

## STEP 3: Summarizing Feature Table and Feature Data

Before running the following commands, the sample-metadata file need to be created. \
<br />
Once the matadata file is ready, we run:
```
qiime feature-table summarize \
  --i-table 18S_table.qza \
  --m-sample-metadata-file 18S_sample-metadata.tsv \
  --o-visualization 18S_table.qzv
```
```
qiime feature-table tabulate-seqs \
  --i-data 18S_rep-seqs.qza \
  --o-visualization 18S_rep-seqs.qzv
```
<br />

---

## STEP4: Taxonomy assignment
<br />

### 18S Eukaryotes data bases:


### Now we preform the classification with a trained classifier:

<br />

*NOTES: Check the following data bases:*
- https://pr2-database.org/
- https://pr2-database.org/post/news/2022-08-01-metapr2/
- https://zenodo.org/record/6896896#.ZBpd4vaZO5c*


```
qiime feature-classifier classify-sklearn 
   --i-classifier ```to be determined```
   --i-reads rep-seqs_16S.qza 
   --o-classification taxonomy_18S_SKLEARN.qza
```

*NOTES: I don't know yet database and classifier to use yet. I'm working on 16S first.*

<br />

---

## STEP5: Phylogenetic tree

This can be made prior to the taxonomic assignment. Generating a phylogenetic tree will allow us to use specific metrics that include a phylogenetic component.  

```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences 18S_rep-seqs.qza \
  --o-alignment 18S_aligned-rep-seqs.qza \
  --o-masked-alignment 18S_masked-aligned-rep-seq.qza \
  --o-tree 18S_unrooted-tree.qza \
  --o-rooted-tree 18S_rooted-tree.qza
```
<br />


---

## STEP6: Analyzing Alpha and Beta diversities

From now one I will prefer doing downstream analyzes in R. But here is the code to generate Alpha and Beta diversity metrics.

### Look at alpha diversity as a function of sequencing depth.

```
qiime diversity alpha-rarefaction \
  --i-table table_16S.qza \
  --i-phylogeny rooted-tree_16S.qza \
  --p-max-depth 8000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization alpha-rarefaction.qzv
```
*Notes: verify what value I should use for ```--p-max-depth```.*

### Compute Alpha and Beta diversity metrics.

By default, the metrics computed are:

Alpha diversity

- Shannon’s diversity index
- Observed OTUs
- Faith’s Phylogenetic Diversity 
- Evenness 

Beta diversity
- Jaccard distance
- Bray-Curtis distance
- unweighted UniFrac distance 
- weighted UniFrac distance


```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny 16S_rooted-tree.qza \
  --i-table 16S_table.qza \
  --p-sampling-depth 1000 \
  --p-n-jobs-or-threads 16 \
  --m-metadata-file 16S_sample-metadata.tsv \
  --output-dir 16S_core-metrics-results
```

*Notes: rarefaction is indicated in```--p-sampling-depth```. Not sure if I will chose to do rarefaction yet. Look at the bibiography.*
<br /><br />


### Testing associations between Alpha diversity metrics and metadata variables. 
<br /> 


- Test associations of categorical variables with Shannon’s diversity index
```
qiime diversity alpha-group-significance \
  --i-alpha-diversity 18S_core-metrics-results/shannon_vector.qza \
  --m-metadata-file 18S_sample-metadata.tsv \
  --o-visualization 18S_core-metrics-results/18S_shannon-group-significance.qzv
```
- Test associations of categorical variables with Evenness
```
qiime diversity alpha-group-significance \
  --i-alpha-diversity 18S_core-metrics-results/evenness_vector.qza \
  --m-metadata-file 18S_sample-metadata.tsv \
  --o-visualization 18S_core-metrics-results/18S_evenness-group-significance.qzv
```
- Test associations of categorical variables with Faith Phylogenetic Diversity
```
qiime diversity alpha-group-significance \
  --i-alpha-diversity 18S_core-metrics-results/faith_pd_vector.qza \
  --m-metadata-file 18S_sample-metadata.tsv \
  --o-visualization 18S_core-metrics-results/18S_faith-pd-group-significance_16S.qzv
```

- Test correlation of continuous variables with Shannon’s diversity index

```
qiime diversity alpha-correlation \
  --i-alpha-diversity 18S_core-metrics-results/shannon_vector.qza \
  --m-metadata-file 18S_sample-metadata.tsv \
  --p-method spearman \
  --o-visualization 18S_core-metrics-results/18S_shannon_correlation.qzv
```
- Test correlation of continuous variables with Evenness
```
qiime diversity alpha-correlation \
  --i-alpha-diversity 18S_core-metrics-results/evenness_vector.qza \
  --m-metadata-file 18S_sample-metadata.tsv \
  --p-method spearman \
  --o-visualization 18S_core-metrics-results/18S_evenness_correlation.qzv
```
- Test correlation of continuous variables with Faith Phylogenetic Diversity
```
qiime diversity alpha-correlation \
  --i-alpha-diversity 18S_core-metrics-results/faith_pd_vector.qza \
  --m-metadata-file 18S_sample-metadata.tsv \
  --p-method spearman \
  --o-visualization 18S_core-metrics-results/18S_faith_pd_correlation.qzv
```

### Testing associations between Beta diversity metrics and metadata variables using PERMANOVA
<br />

*Notes: several parameters to change in the following code. I am far from this now, so I will change these later.*


```
qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics-results_18S/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column transect-name \
  --o-visualization core-metrics-results_18S/unweighted-unifrac-tran-name-significance.qzv \
  --p-pairwise
```

```
qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics-results_18S/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column vegetation \
  --o-visualization core-metrics-results_18S/unweighted-unifrac-subject-group-significance.qzv \
  --p-pairwise
```

### Differential Abundance Analysis
<br />

*Notes: several parameters to change in the following code. I am far from this now, so I will change these later.*

```
qiime feature-table filter-features \
  --i-table table_18S.qza \
  --p-min-frequency 5 \
  --p-min-samples 10 \
  --o-filtered-table 18S_filtered_table.qza
```

```
qiime composition add-pseudocount \
  --i-table filtered_table_18S.qza \
  --o-composition-table 18S_filtered_table_pc.qza
```

```
qiime composition ancom \
  --i-table 18S_filtered_table_pc.qza \
  --m-metadata-file 18S_sample-metadata.tsv \
  --m-metadata-column transect-name \
  --o-visualization 18S_ancom_transect-name.qzv
```

*Notes: now there is ```ancombc``` which includes bias correction, check it*
