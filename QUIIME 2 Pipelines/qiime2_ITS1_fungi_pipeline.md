# QIIME2 Pipeline for ITS1 eDNA Metabarcoding Analysis

---

In this pipeline I will use QIIME2 to perform an analysis of ITS1 reads of soil samples from savanas, pastures, and hurban soils in French Guiana, in order to identify and analyse soil fungi communities. The following pipeline on 16S data is adapted from the physalia course on metbarcoding for microbial ecology (course 30).

---

In this pipeline, it is assumed that QIIME2 was installed correctly. See the QIIME2 16S pipeline for installation instructions.

---



The stop sign &#x1F6D1; will indicate my progress by showing the step that I am at.

---


&#x1F6D1; This pipeline will differ somewhat from the 16S/18S pipeline, notably by using ITSxpress. It is not written yet.

---

## ITS1 Fungi 

```
mkdir ITS1-qiime2
```
```
cd ITS1-qiime2
```
### Activate the conda environment
```
conda activate qiime2-2023.2
```

### Install ITSxpress

```
conda install -c bioconda itsxpress pip install q2-itsxpress
qiime dev refresh-cache
```

---

## STEP1: Import the data, summarize the results, and examine the quality of the reads

qiime tools import \
  --type SampleData[PairedEndSequencesWithQuality] \
  --input-format PairedEndFastqManifestPhred33\
  --input-path manifest.txt \
  --output-path sequences.qza

qiime demux summarize \
  --i-data sequences.qza \
  --o-visualization sequences.qzv
---

## STEP2: Quality control of the sequences and building Feature Table and Feature Data

qiime itsxpress trim-pair-output-unmerged
  --i-per-sample-sequences sequences.qza 
  --p-region ITS1 
  --p-taxa F 
  --p-cluster-id 1.0 
  --o-trimmed trimmed.qza