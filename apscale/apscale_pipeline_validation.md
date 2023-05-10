# Apscale Pipeline Validation

We assume that Apscale and dependencies are installed.

---

This script is an attempt to work through the Apscale pipeline entirely, understanding each steps, with a subset of data to make the computationally intensive tasks faster and validating the methods used before working the full datasets. Here, there won't be detailed explanation of each steps, but rather exploring and discussing the outputs.
<br />

---

The stop sign &#x1F6D1; will indicate my progress by showing the step that I am at.

---

APSCALE produces OTUs and ESVs in an automated pipeline with parameters that can be changed in an settings excel file. The structure of an APSCALE project is the following: 

```
C:\USERS\DOMINIK\DESKTOP\EXAMPLE_PROJECT
├───1_raw data
│   └───data
├───2_demultiplexing
│   └───data
├───3_PE_merging
│   └───data
├───4_primer_trimming
│   └───data
├───5_quality_filtering
│   └───data
├───6_dereplication_pooling
│   └───data
│       ├───dereplication
│       └───pooling
├───7_otu_clustering
│   └───data
├───8_denoising
│   └───data
└───9_lulu_filtering
    ├───denoising
    │   └───data
    └───otu_filtering
        └───data
```

Visite APSCALE github page for more information: https://github.com/DominikBuchner/apscale

We will create a project for the pipeline validation: 
```
apscale --create_project apscale_pipeline_validation
```

## Step 1: Demultiplex raw data.

APSCALE does not handle the demultiplexing of the raw data. We will use cutadapt for this step.

Copy the raw data subset into the ```1_raw/data/``` folder. Here demultiplex with the wanted package.

Next, move the demultiplex data into the ```2_demultiplexing/data/``` folder. 

## Step 2: Setting the parameters.

The ```Setting.xlsx``` file is where parameters for all the following steps are set. It is possible to run all the following steps at once, or run the steps one by one. 


