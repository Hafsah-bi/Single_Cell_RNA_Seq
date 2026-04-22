# Single_Cell_RNA_Seq
This repository organizes learning materials and practical references for single-cell RNA-seq into three core sections: 10x data pre-processing, basic scRNA-seq analysis, and AnnData fundamentals.

---

##  Repository Structure

```Single_Cell_RNA_Seq-pipeline/
├── 1_preprocessing_10X_Single_Cell_RNA/        # Stage 1
├── 2_basic_scrna_seq_tutorial/        # Stage 2
├── 3_AnnData for Single-Cell Analysis/ # Stage 3
└── anndata1
└── anndata2
└── README.md
```

---


# 1. Pre-processing of 10x Single-Cell RNA Datasets

## Introduction: Understand the 10x Genomics Workflow

This repository presents a concise step-by-step guide for pre-processing **10x Genomics single-cell RNA-seq data**, following the Galaxy Training Network tutorial:

- [Pre-processing of 10x Single-Cell RNA Datasets](https://training.galaxyproject.org/training-material/topics/single-cell/tutorials/scrna-preprocessing-tenx/tutorial.html)

In the 10x Genomics workflow, sequencing reads are labeled using **cell barcodes** and **unique molecular identifiers (UMIs)**, allowing transcript counts to be assigned to individual cells. The purpose of pre-processing is to convert raw sequencing data into a structured **count matrix**, inspect mapping and quantification results, and derive a filtered matrix containing high-quality cells for downstream single-cell analysis.

---

## Step 1: Producing a Count Matrix from FASTQ

This stage converts sequencing reads into an initial expression matrix suitable for inspection and filtering.

### a. Data upload and organization

Create a new Galaxy history and rename it, for example:

- `scRNA-seq 10X dataset tutorial`

Then import the required input files from Zenodo or the Galaxy data library.

### Input files

| **File type** | **File** | **Link** |
|---|---|---|
| FASTQ | `subset_pbmc_1k_v3_S1_L001_R1_001.fastq.gz` | [Download](https://zenodo.org/record/3457880/files/subset_pbmc_1k_v3_S1_L001_R1_001.fastq.gz) |
| FASTQ | `subset_pbmc_1k_v3_S1_L001_R2_001.fastq.gz` | [Download](https://zenodo.org/record/3457880/files/subset_pbmc_1k_v3_S1_L001_R2_001.fastq.gz) |
| FASTQ | `subset_pbmc_1k_v3_S1_L002_R1_001.fastq.gz` | [Download](https://zenodo.org/record/3457880/files/subset_pbmc_1k_v3_S1_L002_R1_001.fastq.gz) |
| FASTQ | `subset_pbmc_1k_v3_S1_L002_R2_001.fastq.gz` | [Download](https://zenodo.org/record/3457880/files/subset_pbmc_1k_v3_S1_L002_R2_001.fastq.gz) |
| Gene annotation | `Homo_sapiens.GRCh37.75.gtf` | [Download](https://zenodo.org/record/3457880/files/Homo_sapiens.GRCh37.75.gtf) |
| Barcode whitelist | `3M-february-2018.txt.gz` | [Download](https://zenodo.org/record/3457880/files/3M-february-2018.txt.gz) |

### Notes
- Create and rename a new history before uploading files.
- Import data directly via Zenodo links or from the Galaxy data library.
- At the end of this step, the history should contain **4 FASTQ files**, **1 GTF file**, and **1 barcode whitelist file**.

Proper file organization is necessary to ensure that downstream quantification tools interpret read structure and sample identity correctly.

---

## Step 2: Performing the Demultiplexing and Quantification

This stage assigns reads to cellular barcodes and transcripts, then generates output files summarizing alignment and quantification results.

### a. RNA STARsolo

Run **RNA STARsolo** in Galaxy using the following configuration to perform barcode-aware alignment and quantification.

### Tool
- **RNA STARsolo** (`Galaxy version 2.7.11a+galaxy1`)

### Parameters

| **Setting** | **Value** |
|---|---|
| Custom or built-in reference genome | Use a built-in index |
| Reference genome with or without an annotation | Use genome reference without builtin gene-model |
| Select reference genome | Human (*Homo sapiens*): hg19 Full or Human (*Homo sapiens*) (b37): hg19 |
| Gene model (gff3, gtf) file for splice junctions | `Homo_sapiens.GRCh37.75.gtf` |
| Length of genomic sequence around annotated junctions | `100` |
| Type of single-cell RNA-seq | Drop-seq or 10X Chromium |
| Input Type | Separate barcode and cDNA reads |
| RNA-Seq FASTQ/FASTA file, Barcode reads | `L001_R1_001`, `L002_R1_001` |
| RNA-Seq FASTQ/FASTA file, cDNA reads | `L001_R2_001`, `L002_R2_001` |
| RNA-Seq Cell Barcode Whitelist | `3M-february-2018.txt.gz` |
| Configure Chemistry Options | Chromium chemistry v3 |
| UMI deduplication (collapsing) algorithm | CellRanger2-4 algorithm |
| Type of UMI filtering | Remove UMIs with N and homopolymers |
| Matching the Cell Barcodes to the WhiteList | Multiple matches (CellRanger 2, 1MM_multi) |
| Strandedness of Library | Read strand same as the original RNA molecule |
| Collect UMI counts for these genomic features | Gene: Count reads matching the Gene Transcript |
| Cell filter type and parameters | Do not filter |
| Field 3 in the Genes output | Gene Expression |

### Input selection
Use multi-select for the FASTQ files:

- **Barcode reads:** `L001_R1_001`, `L002_R1_001`
- **cDNA reads:** `L001_R2_001`, `L002_R2_001`

### Purpose
This step performs:

- alignment to the human reference genome,
- barcode and UMI processing,
- transcript quantification,
- generation of an initial unfiltered count matrix for downstream quality assessment.

### b. Inspecting the Output Files

RNA STARsolo produces six main outputs:

| **Output** | **Description** |
|---|---|
| Log | Program log and mapping statistics |
| Feature Statistic Summaries | Barcode and quantification metrics |
| Alignments | BAM file of mapped reads |
| Matrix Gene Counts | Count matrix in Matrix Market format |
| Barcodes | Cell barcode list |
| Genes | Gene list |

The **log** and **feature summaries** files are the main sources for quality assessment. The matrix, barcode, and gene files together form the unfiltered count matrix.

#### i. Mapping Quality

Use **MultiQC** to inspect the STAR log.

**Tool:** `MultiQC (Galaxy version 1.27+galaxy0)`

| **Setting** | **Value** |
|---|---|
| Which tool was used generate logs? | STAR |
| Type of STAR output? | Log |
| STAR log output | `RNA STARsolo: log` |

**Result:**  
- **Uniquely mapped reads:** `87.5%`

This indicates good alignment quality.

#### ii. Quantification Quality

Inspect the **Feature Statistic Summaries** file directly in Galaxy.

Key metric groups:

| **Metric** | **Meaning** |
|---|---|
| Barcode mismatch metrics | Reads with barcodes not matching the whitelist |
| `noUnmapped + MultiFeature` | Reads without clear feature assignment |
| `yessubWLmatch_UniqueFeature` | Reads successfully counted |

![RNA STARsolo Log Output](images/1_RNA_STARSolo_Log_Output.png)
> *Figure 1: RNA STARsolo feature statistic summary showing barcode matching and gene assignment metrics, including uniquely assigned reads, reads without annotated features, and the number of detected cell barcodes.*

**Key observations**
- **Detected cells (`yesCellBarcodes`)**: ~`5200`
- **Largest category**: `yessubWLmatch_UniqueFeature`
- **`noNoFeature`** reads may be relatively high and are generally expected for this dataset

At this stage, confirm that most reads map well, most usable reads are assigned to features, and the detected barcode count is reasonable.

---

## Step 3: Producing a Quality Count Matrix

The RNA STARsolo output matrix is provided in bundled 10x format (`matrix.mtx`, `genes.tsv`, `barcodes.tsv`). While this format is compatible with downstream single-cell workflows, it includes many low-quality or background barcodes. To generate a more representative count matrix, filter barcodes with **DropletUtils**.

### a. Cell Ranger-like filtering

Use **DropletUtils** to emulate the default Cell Ranger-style barcode filtering approach.

**Tool**
- **DropletUtils** (`Galaxy version 1.10.0+galaxy2`)

| **Setting** | **Value** |
|---|---|
| Format for the input matrix | Bundled (`barcodes.tsv`, `genes.tsv`, `matrix.mtx`) |
| Count Data | `Matrix Gene Counts` |
| Genes List | `Genes` |
| Barcodes List | `Barcodes` |
| Operation | Filter for Barcodes |
| Method | DefaultDrops |
| Expected Number of Cells | `3000` |
| Upper Quantile | `0.99` |
| Lower Proportion | `0.1` |
| Format for output matrices | Bundled (`barcodes.tsv`, `genes.tsv`, `matrix.mtx`) |

### b. Introspective method

For a more data-driven approach, first inspect barcode distributions and then apply custom filtering.

#### i. Rank barcodes

Generate a barcode rank plot to visualize total UMI counts across barcodes and help identify the transition between real cells and background droplets.

**Tool**
- **DropletUtils** (`Galaxy version 1.10.0+galaxy2`)

| **Setting** | **Value** |
|---|---|
| Format for the input matrix | Bundled (`barcodes.tsv`, `genes.tsv`, `matrix.mtx`) |
| Count Data | `Matrix Gene Counts` |
| Genes List | `Genes` |
| Barcodes List | `Barcodes` |
| Operation | Rank Barcodes |
| Lower Bound | `100` |

![Barcode Rank Plot](images/2_Barcode_Ranks.png)
> *Figure 2: Barcode rank plot showing total UMI counts across ranked barcodes. The knee and inflection points indicate the approximate thresholds separating high-RNA cell-containing droplets from low-count empty droplets, providing a rough estimate of the expected number of cells in the sample.*

#### ii. Custom filtering

Apply **EmptyDrops** for barcode filtering based on statistical significance.

**Tool**
- **DropletUtils** (`Galaxy version 1.10.0+galaxy2`)

| **Setting** | **Value** |
|---|---|
| Format for the input matrix | Bundled (`barcodes.tsv`, `genes.tsv`, `matrix.mtx`) |
| Count Data | `Matrix Gene Counts` |
| Genes List | `Genes` |
| Barcodes List | `Barcodes` |
| Operation | Filter for Barcodes |
| Method | EmptyDrops |
| Lower-bound Threshold | `200` |
| FDR Threshold | `0.01` |
| Format for output matrices | Bundled (`barcodes.tsv`, `genes.tsv`, `matrix.mtx`) |

![Detected Cells Plot](images/3_Detected_Cells.png)
> *Figure 3: EmptyDrops detected-cells plot showing total UMI count versus negative log probability for each barcode. Barcodes with stronger evidence of deviating from the ambient RNA background are identified as likely real cells.*

### Notes
- The bundled matrix is highly sparse because it includes many low-count barcodes.
- **DefaultDrops** is a simple Cell Ranger-like option.
- **Rank Barcodes + EmptyDrops** provides a more flexible and interpretable filtering strategy.
- The filtered output can be used as a higher-quality input for downstream single-cell analysis.

---

## Summary of Workflow

The complete preprocessing procedure can be understood in three major steps:

1. **Produce an initial count matrix from FASTQ files**
2. **Inspect demultiplexing and quantification outputs**
3. **Generate a quality-controlled count matrix**

These steps establish the foundation for downstream single-cell RNA-seq analyses such as normalization, clustering, and cell-type annotation.

---

# 2. Basic scRNA-seq analysis

This repository contains a Python notebook that walks through a complete **single-cell RNA-seq (scRNA-seq)** analysis workflow using **Scanpy**, **CellTypist**, and **decoupler**.

The dataset consists of **bone marrow mononuclear cells from healthy human donors**. The workflow covers the major stages of a standard scRNA-seq analysis pipeline, including:

- Data loading
- Quality control
- Filtering
- Doublet detection
- Normalization
- Feature selection
- Dimensionality reduction
- Clustering
- Visualization
- Cell-type annotation
- Marker gene discovery

The notebook is designed as a practical end-to-end tutorial for preprocessing and interpreting scRNA-seq data in Python.

---

##  Dependencies
 
| Package | Purpose |
|---|---|
| `scanpy` | Core scRNA-seq analysis |
| `anndata` | Data structure (AnnData) |
| `pooch` | Data download/caching |
| `scrublet` | Doublet detection |
| `leidenalg` / `igraph` | Graph-based clustering |
| `celltypist` | Automatic cell-type annotation |
| `decoupler` | Enrichment-based annotation |
| `omnipath` | PanglaoDB marker database |

---

## Preprocessing and clustering

The data used in this tutorial contains **8,785 cells** and **36,601 measured genes**.

This notebook demonstrates a basic preprocessing and clustering workflow on bone marrow mononuclear cells collected from healthy human donors. The samples were measured using the **10X Multiome Gene Expression and Chromatin Accessibility kit** and were part of the **OpenProblems NeurIPS 2021 benchmarking dataset**.

### Tools used

- **anndata** — storage and manipulation of annotated single-cell matrices
- **scanpy** — preprocessing, visualization, clustering, and differential expression
- **pooch** — dataset download and caching
- **scrublet** — doublet detection
- **celltypist** — automated cell-type annotation
- **decoupler** — enrichment-based annotation and activity scoring

---

## Library Import and Plot Settings

The notebook begins by importing the required Python libraries and configuring Scanpy plotting defaults.

### Main steps

- Import `anndata`, `pooch`, and `scanpy`
- Configure plotting parameters with a white background and adjusted DPI for cleaner figures

---

## Data Download and Dataset Construction

This section downloads the input data and constructs a combined `AnnData` object for downstream analysis.

### Main steps

- Download sample `.h5` files using `pooch`
- Read each sample using `scanpy.read_10x_h5`
- Make variable names unique
- Concatenate multiple samples into a single dataset
- Store sample labels in metadata
- Ensure observation names are unique

---

## Quality Control

Quality control metrics are computed to assess cell quality and identify potentially problematic cells before downstream analysis.

### QC metrics calculated

- Number of genes detected per cell
- Total counts per cell
- Percentage of counts from:
  - Mitochondrial genes
  - Ribosomal genes
  - Hemoglobin genes

### Main steps

- Label mitochondrial genes using the `MT-` prefix
- Label ribosomal genes using `RPS` and `RPL`
- Label hemoglobin genes using a pattern match
- Compute QC metrics with `scanpy.pp.calculate_qc_metrics`

---

## Quality Control Visualization

This section generates plots to inspect QC distributions and identify outlier cells.

### Visualizations included

- Violin plots for:
  - `n_genes_by_counts`
  - `total_counts`
  - `pct_counts_mt`
- Scatter plot of:
  - `total_counts` vs `n_genes_by_counts`
  - Colored by mitochondrial percentage

These plots help identify:

- Low-quality cells
- Potentially stressed or dying cells
- Cells with unusual library sizes or gene counts

---

## Cell and Gene Filtering

Basic filtering is applied to remove low-information cells and rarely detected genes.

### Filtering strategy

- Remove cells expressing fewer than **100 genes**
- Remove genes detected in fewer than **3 cells**

This notebook uses a relatively permissive first-pass filtering approach so that stricter QC decisions can be revisited later after clustering.

---

## Doublet detection

Doublets are droplets that contain more than one cell, which can distort downstream analysis if not removed.

This notebook uses **Scrublet** through Scanpy to detect likely doublets.

### Main steps

- Install and run `scrublet`
- Detect doublets separately for each sample using `batch_key="sample"`
- Store:
  - `doublet_score`
  - `predicted_doublet`

### Notes

Alternative doublet detection tools in the scverse ecosystem include:

- DoubletDetection
- SOLO

---

## Normalization

The notebook applies count-depth normalization followed by log transformation.

### Main steps

- Save the original count matrix in `adata.layers["counts"]`
- Normalize counts across cells using `scanpy.pp.normalize_total`
- Log-transform expression values using `scanpy.pp.log1p`

### Why this matters

This step:

- Adjusts for differences in sequencing depth
- Reduces skewness in count distributions
- Prepares the data for PCA, clustering, and visualization

---

## Feature selection

Feature selection identifies genes with the most informative biological variation.

### Main steps

- Compute highly variable genes with `scanpy.pp.highly_variable_genes`
- Select the top **2,000 highly variable genes**
- Account for batch structure using `batch_key="sample"`
- Visualize selected genes with `scanpy.pl.highly_variable_genes`

---

## Dimensionality Reduction

Dimensionality reduction compresses the dataset into a smaller number of informative components.

### Main steps

- Run PCA using `scanpy.tl.pca`
- Inspect explained variance with `scanpy.pl.pca_variance_ratio`

### Why this matters

PCA:

- Denoises the data
- Captures major axes of variation
- Provides the representation used for neighborhood graph construction

---

## Neighborhood Graph and UMAP Embedding

Cells are connected based on transcriptional similarity and projected into a two-dimensional space for visualization.

### Main steps

- Build the k-nearest neighbor graph with `scanpy.pp.neighbors`
- Compute a UMAP embedding with `scanpy.tl.umap`
- Visualize sample mixing with `scanpy.pl.umap(color="sample")`

### Interpretation

This step helps assess:

- Overall dataset structure
- Cluster separation
- Possible batch effects across samples

---

## Clustering

The notebook uses the **Leiden algorithm** to cluster cells based on the neighborhood graph.

### Main steps

- Install `leidenalg` and `igraph`
- Run Leiden clustering with `scanpy.tl.leiden`
- Store cluster labels in `adata.obs["leiden"]`
- Visualize clusters on UMAP

### Outcome

This identifies groups of cells with similar transcriptional profiles, providing the basis for downstream annotation.

---

## Re-assess quality control and cell filtering

After clustering, QC is revisited to identify whether some clusters are enriched for likely doublets or poor-quality cells.

### Main steps

- Convert `predicted_doublet` to categorical format
- Plot:
  - `leiden`
  - `predicted_doublet`
  - `doublet_score`
- Remove cells predicted as doublets
- Re-visualize QC metrics on UMAP:
  - `log1p_total_counts`
  - `pct_counts_mt`
  - `log1p_n_genes_by_counts`

### Why this matters

This second-pass QC step helps refine the dataset using both technical metrics and cluster context.

---

## Cell-type annotation

Cell-type annotation is performed using both **manual marker-based inspection** and **automated methods**.

### Installed annotation tools

- **CellTypist**
- **decoupler**

Annotation is approached in multiple ways to increase robustness and biological interpretability.

---

## Multi-Resolution Clustering

Several Leiden resolutions are computed to compare clustering granularity.

### Main steps

- Compute clustering at:
  - `resolution=0.02`
  - `resolution=0.5`
  - `resolution=2`
- Store results in:
  - `leiden_res0_02`
  - `leiden_res0_5`
  - `leiden_res2`
- Compare these solutions visually on UMAP

### Purpose

This helps determine a clustering resolution that balances:

- Biological interpretability
- Separation of distinct cell populations
- Avoidance of over-clustering

---

## Marker gene set

A manually curated set of marker genes is defined for major immune and erythroid populations.

### Example cell types included

- CD14+ Mono
- CD16+ Mono
- cDC2
- Erythroblast
- Proerythroblast
- NK
- ILC
- Naive CD20+ B
- B cells
- Plasma cells
- Plasmablast
- CD4+ T
- CD8+ T
- T naive
- pDC

### Purpose

These marker sets serve as a reference for interpreting cluster identities through expression patterns.

---

## Marker-Based Cluster Visualization

Marker expression is visualized across clustering solutions using dot plots.

### Main steps

- Plot marker genes grouped by `leiden_res0_02`
- Plot marker genes grouped by `leiden_res0_5`

### Goal

This helps determine which clustering resolution best separates biologically meaningful cell populations.

---

## Automatic label prediction

Automated annotation is performed using **CellTypist**.

### Download CellTypist model

The notebook downloads the **Immune_All_Low.pkl** reference model.

### Predict cell type labels

### Main steps

- Load the CellTypist model
- Run annotation with:
  - `majority_voting=True`
  - `over_clustering="leiden_res0_5"`
- Convert predictions back into `AnnData`
- Visualize predicted labels on UMAP

### Note

CellTypist expects log1p-normalized expression scaled to **10,000 counts per cell**, so deviations from that preprocessing may affect prediction accuracy.

---

## Annotation with enrichment analysis

As a complementary strategy, the notebook uses **decoupler** together with **PanglaoDB** marker genes for enrichment-based annotation.

### Main steps

- Install and import `omnipath`
- Retrieve PanglaoDB markers using `decoupler`
- Filter markers for relevant entries
- Remove duplicates
- Run multivariate linear modeling with `dc.run_mlm`
- Store enrichment results in `adata.obsm`

### Outputs

The enrichment analysis generates activity scores representing marker enrichment for different cell types across cells.

---

## Differentially-expressed Genes as Markers

Cluster-specific marker genes are identified using differential expression analysis.

### Main steps

- Run `scanpy.tl.rank_genes_groups`
- Filter marker genes by fold change
- Visualize the top markers in a dot plot

### Why this matters

Differential expression helps validate cluster identity and discover genes that distinguish one cluster from others.

---

## Cluster-specific marker visualization

The notebook further inspects marker genes from a selected cluster using UMAP and violin plots.

### Main steps

- Select representative genes from cluster 3:
  - `LYZ`
  - `ACTB`
  - `S100A6`
  - `S100A4`
  - `CST3`
- Plot their expression on UMAP
- Plot violin plots across clusters

### Purpose

These plots help confirm whether the identified genes are specifically enriched in the selected cluster and support its biological annotation.

---

## Output

By the end of the notebook, the workflow provides:

- A filtered and normalized scRNA-seq dataset
- Cluster assignments
- UMAP visualizations
- Doublet predictions
- Cell-type annotations from CellTypist
- Enrichment-based annotations from decoupler
- Differentially expressed marker genes for each cluster

---

# 3.  AnnData for Single-Cell Analysis
 
A two-part tutorial series on working with **AnnData** — the core data structure for single-cell omics analysis in Python. These notebooks walk through building, annotating, subsetting, and exploring AnnData objects, from scratch construction to real-world dataset inspection.
 
---
 
## Overview
 
[AnnData](https://anndata.readthedocs.io/) is the standard Python container for single-cell datasets. It stores a gene expression matrix (cells × genes) together with cell metadata, gene metadata, embeddings, alternate data representations, and unstructured results — all in one organized object, and serializable to `.h5ad` format.
 
This series is suitable for beginners getting started with single-cell Python tooling, as well as intermediate users looking to solidify their understanding of how AnnData manages data internally.
 
---
 
## Requirements
 
```bash
pip install anndata scanpy numpy pandas scipy pooch
```
 
| Package | Purpose |
|---|---|
| `anndata` | Core data structure |
| `scanpy` | Single-cell analysis & plotting utilities |
| `numpy` | Numerical operations |
| `pandas` | Metadata tables |
| `scipy` | Sparse matrix support |
| `pooch` | Dataset downloading & caching |
 
---
 
## Part A — Introduction to AnnData
 
**Notebook:** `anndata1.ipynb`
 
This notebook builds an AnnData object from scratch using simulated data, demonstrating the full anatomy of the object and its core operations.
 
### Topics Covered
 
**1. Initializing AnnData**
- Creating a sparse count matrix with `csr_matrix` and `np.random.poisson`
- Wrapping it in an `AnnData` object
- Accessing the main matrix via `adata.X`
  
**2. Naming Observations and Variables**
- Setting `adata.obs_names` (cell IDs) and `adata.var_names` (gene IDs)
- Label-based slicing, similar to pandas
  
**3. Cell and Gene Annotations**
- Adding cell-type labels to `adata.obs` using `pd.Categorical`
- Filtering cells by annotation (e.g., selecting only B cells)
- Rebuilding an AnnData object with a full metadata DataFrame
  
**4. Multi-dimensional Annotations**
- Storing UMAP coordinates in `adata.obsm`
- Storing gene-level matrices in `adata.varm`
  
**5. Unstructured Metadata**
- Using `adata.uns` for results that don't fit row- or column-aligned tables
  
**6. Layers**
- Storing log-transformed data alongside raw counts in `adata.layers`
- Exporting a layer as a pandas DataFrame with `.to_df()`
  
**7. Saving and Loading**
- Writing to `.h5ad` with gzip compression using `adata.write()`
- Inspecting HDF5 contents with `h5ls`
- Opening large files in backed mode (`backed='r'`) to avoid loading everything into RAM
- Checking backed status with `adata.isbacked` and `adata.filename`
  
**8. Views vs. Copies**
- Understanding when a subset is a view vs. an independent object
- Using `.copy()` to avoid unintended side effects
- How editing `.obs` on a view triggers `ImplicitModificationWarning` and materializes the view
  
**9. Metadata Queries**
- Filtering by multiple values with `.isin()`
- Conditional row selection using boolean masks on `.obs`
  
---
 
## Part B — Understanding the AnnData Object
 
**Notebook:** `anndata2.ipynb`
 
This notebook uses a real processed PBMC dataset (3k cells) from the scverse tutorial repository to explore the full structure of a production AnnData object.
 
### Topics Covered
 
**1. Loading a Real Dataset**
- Downloading a processed `.h5ad` file with `pooch` (with hash verification)
- Reading it with `anndata.read_h5ad()`
  
**2. Exploring the Expression Matrix**
- Inspecting `adata.X` (log-normalized counts in sparse format)
- Examining non-zero values (`adata.X.data`), their positions (`adata.X.indices`), and sparsity fraction
  
**3. Working with Layers**
- Listing available layers with `adata.layers`
- Accessing raw counts via `adata.layers["raw"]`
- Creating a new CPM (counts-per-million) normalized layer using `sc.pp.normalize_total()`
- Comparing layers visually with `sc.pl.matrixplot()` for marker genes (CD8A, CD4, KLRB1)
  
**4. Cell Metadata with `.obs`**
- Inspecting the full `.obs` DataFrame
- Accessing and counting cells by type (e.g., B cells)
- Adding a quality flag column (`is_low_quality`) based on mitochondrial percentage
- Filtering out low-quality cells with boolean masking
  
**5. Gene Metadata with `.var`**
- Inspecting the `.var` DataFrame
- Replacing `var_names` with gene IDs from a metadata column
- Subsetting genes by name using `.isin()` on `.var`
  
**6. Unstructured Metadata with `.uns`**
- Listing `.uns` keys
- Accessing stored Louvain clustering parameters and colors
- Inspecting PCA variance explained
  
**7. Embeddings with `.obsm`**
- Iterating over `.obsm` keys and shapes
- Visualizing PCA, t-SNE, and UMAP embeddings side by side
- Highlighting B cells with color mapping
  
**8. Pairwise Matrices with `.obsp`**
- Accessing pairwise cell distances in `adata.obsp["distances_all"]`
- Visualizing the distance matrix as a heatmap
- Reordering cells by Louvain cell type to reveal block structure
  
**9. Views, Copies, and Object Materialization**
- Demonstrating that subsets are initially views linked to parent data
- Showing that parent modifications propagate to views
- Using `.copy()` to create a fully independent object
- Triggering view materialization by editing `.obs`, and confirming divergence from the parent
  
---
 
## Key Concepts
 
| Slot | Type | Stores |
|---|---|---|
| `adata.X` | sparse/dense matrix | Main expression matrix (cells × genes) |
| `adata.obs` | `pd.DataFrame` | Per-cell metadata (cell type, QC flags, etc.) |
| `adata.var` | `pd.DataFrame` | Per-gene metadata (gene IDs, highly variable, etc.) |
| `adata.obsm` | dict of arrays | Multi-dimensional cell annotations (PCA, UMAP, t-SNE) |
| `adata.varm` | dict of arrays | Multi-dimensional gene annotations |
| `adata.obsp` | dict of matrices | Pairwise cell matrices (distances, connectivities) |
| `adata.layers` | dict of matrices | Alternate expression representations (raw, normalized) |
| `adata.uns` | dict | Unstructured metadata (parameters, colors, results) |
 
### View vs. Copy
 
```python
# View — linked to the parent object
adata_view = adata[:5, :10]
 
# Copy — fully independent
adata_copy = adata[:5, :10].copy()
```
 
A view becomes a real object automatically when you modify its `.obs` or other metadata. After materialization, changes to the parent no longer affect it.
 
---

## All Tutorial Sources

| Stage | Tutorial | URL |
|-------|----------|-----|
| 1 | GTN: Hands-on Pre-processing of 10X scRNA Datasets | https://training.galaxyproject.org/training-material/topics/single-cell/tutorials/scrna-preprocessing-tenx/tutorial.html |
| 2 | scverse: basic-scrna-tutorial.ipynb | https://github.com/scverse/scanpy-tutorials/blob/main/basic-scrna-tutorial.ipynb |
| 3a | AnnData Getting Started (anndata docs v0.13) | https://anndata.readthedocs.io/en/latest/tutorials/notebooks/getting-started.html |
| 3b | AnnData Getting Started (scverse-tutorials) | https://scverse-tutorials.readthedocs.io/en/latest/notebooks/anndata_getting_started.html |

---


