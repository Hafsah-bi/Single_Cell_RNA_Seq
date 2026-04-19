# Single_Cell_RNA_Seq
This repository organizes learning materials and practical references for single-cell RNA-seq into three core sections: 10x data pre-processing, basic scRNA-seq analysis, and AnnData fundamentals.

# Pre-processing of 10x Single-Cell RNA Datasets

## Introduction: Understand the 10x Genomics Workflow

This repository presents a concise step-by-step guide for pre-processing **10x Genomics single-cell RNA-seq data**, following the Galaxy Training Network tutorial:

- [Pre-processing of 10x Single-Cell RNA Datasets](https://training.galaxyproject.org/training-material/topics/single-cell/tutorials/scrna-preprocessing-tenx/tutorial.html)

In the 10x Genomics workflow, sequencing reads are labeled using **cell barcodes** and **unique molecular identifiers (UMIs)**, allowing transcript counts to be assigned to individual cells. The purpose of pre-processing is to convert raw sequencing data into a structured **count matrix**, inspect mapping and quantification results, and derive a filtered matrix containing high-quality cells for downstream single-cell analysis.

---

## Step 1: Producing a Count Matrix from FASTQ

This stage converts sequencing reads into an initial expression matrix suitable for inspection and filtering.

### a. Data upload and organization

At the beginning of the workflow, the required input files are uploaded and organized in the analysis environment. These commonly include:

- **FASTQ files** generated from sequencing
- Reference files required for alignment and quantification
- Associated metadata, where applicable

Proper file organization is necessary to ensure that downstream quantification tools interpret read structure and sample identity correctly.

---

## Step 2: Performing the Demultiplexing and Quantification

This stage assigns reads to cellular barcodes and transcripts, then generates output files summarizing alignment and quantification results.

### a. Inspecting the Output Files

Following quantification, the resulting files should be examined carefully to assess data quality and interpretability.

#### i. Mapping Quality

Mapping quality evaluation focuses on how effectively reads align to the reference.

Key aspects include:

- Proportion of reads successfully mapped
- Fraction of uniquely assigned reads
- General consistency of alignment outcomes

These metrics help determine whether the sequencing data and reference preparation are suitable for reliable downstream analysis.

#### ii. Quantification Quality

Quantification quality concerns the reliability of expression estimates across cell barcodes.

Typical points of inspection include:

- Distribution of counts across barcodes
- Number of detected genes
- UMI-based transcript counting patterns
- Identification of potential empty droplets or low-information barcodes

This step provides the basis for selecting high-confidence cellular observations.

---

## Step 3: Producing a Quality Count Matrix

The final stage refines the initial matrix to retain barcodes that most likely represent genuine cells.

### a. Cell Ranger Method

The **Cell Ranger method** applies the standard 10x Genomics approach for identifying valid cells and generating a filtered count matrix. This method provides an established default strategy for selecting cell-associated barcodes.

### b. Introspective Method

The **introspective method** applies manual or semi-manual evaluation of barcode distributions to derive a quality-controlled matrix.

#### i. Rank Barcodes

Barcode ranking is used to visualize count distributions and distinguish likely cells from background signal. This enables inspection of barcode abundance patterns before applying filtering criteria.

#### ii. Custom Filtering

Custom filtering allows the analyst to define thresholds according to dataset-specific characteristics.

This may involve:

- Removing low-count barcodes
- Excluding poor-quality observations
- Retaining barcodes supported by expected count and feature profiles

This approach is useful when standard filtering does not fully capture the structure or quality characteristics of the dataset.

---

## Summary of Workflow

The complete preprocessing procedure can be understood in three major steps:

1. **Produce an initial count matrix from FASTQ files**
2. **Inspect demultiplexing and quantification outputs**
3. **Generate a quality-controlled count matrix**

These steps establish the foundation for downstream single-cell RNA-seq analyses such as normalization, clustering, and cell-type annotation.

---

## Reference

This repository is based on the following tutorial:

- [Galaxy Training Network: Pre-processing of 10x Single-Cell RNA Datasets](https://training.galaxyproject.org/training-material/topics/single-cell/tutorials/scrna-preprocessing-tenx/tutorial.html)
