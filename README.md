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

## Reference

This repository is based on the following tutorial:

- [Galaxy Training Network: Pre-processing of 10x Single-Cell RNA Datasets](https://training.galaxyproject.org/training-material/topics/single-cell/tutorials/scrna-preprocessing-tenx/tutorial.html)

# Basic scRNA-seq analysis

A step-by-step single-cell RNA sequencing (scRNA-seq) analysis pipeline using **Scanpy**, covering quality control, normalization, dimensionality reduction, clustering, and cell-type annotation.

---

## 📁 Notebook

`basic-scrna-tutorial_updated.ipynb`

---
 
## 📋 Overview
 
This tutorial walks through a complete scRNA-seq analysis workflow on **bone marrow mononuclear cells** from healthy human donors (OpenProblems NeurIPS 2021 benchmarking dataset), measured with the **10X Multiome Gene Expression and Chromatin Accessibility kit**.
 
**Dataset:** 8,785 cells × 36,601 genes across 2 samples (`s1d1`, `s1d3`)
 
---
 
## 🔧 Installation
 
```bash
pip install anndata scanpy pooch scrublet leidenalg celltypist decoupler omnipath
pip install --upgrade scanpy igraph leidenalg
```
 
---
 
## 📦 Dependencies
 
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
 
## 🚀 Workflow
 
### Step 1 — Load Data
 
Download and concatenate two 10X Multiome samples into a single AnnData object.
 
```python
import anndata as ad
import scanpy as sc
import pooch

sc.set_figure_params(dpi=50, facecolor="white")

def download_sample(sample_id, known_hash):
    path = pooch.retrieve(
        path=pooch.os_cache("scverse_tutorials"),
        url=f"https://exampledata.scverse.org/tutorials/neurips-2021/{sample_id}_filtered_feature_bc_matrix.h5",
        known_hash=known_hash,
    )
    sample_adata = sc.read_10x_h5(path)
    sample_adata.var_names_make_unique()
    return sample_adata
 
samples = {
    "s1d1": "md5:a99285913ea3f3d22600d3d2f8a88e34",
    "s1d3": "md5:825f7f7578e3dc0b8955f5a97a402338",
}
adatas = {id_: download_sample(id_, h) for id_, h in samples.items()}
adata = ad.concat(adatas, label="sample")
adata.obs_names_make_unique()
```
 
---
 
### Step 2 — Quality Control
 
Annotate mitochondrial, ribosomal, and hemoglobin genes, then calculate QC metrics.
 
```python
adata.var["mt"]   = adata.var_names.str.startswith("MT-")
adata.var["ribo"] = adata.var_names.str.startswith(("RPS", "RPL"))
adata.var["hb"]   = adata.var_names.str.contains("^HB[^(P)]")
 
sc.pp.calculate_qc_metrics(adata, qc_vars=["mt", "ribo", "hb"], inplace=True, log1p=True)
 
# Visualize QC metrics
sc.pl.violin(adata, ["n_genes_by_counts", "total_counts", "pct_counts_mt"], jitter=0.4, multi_panel=True)
sc.pl.scatter(adata, "total_counts", "n_genes_by_counts", color="pct_counts_mt")
 
# Permissive filtering
sc.pp.filter_cells(adata, min_genes=100)
sc.pp.filter_genes(adata, min_cells=3)
```
 
> **Tip:** For multi-batch datasets, perform QC per sample individually, as thresholds can vary substantially between batches.
 
---
 
### Step 3 — Doublet Detection
 
Detect and flag cell doublets using Scrublet.
 
```python
sc.external.pp.scrublet(adata, batch_key="sample")
# Adds `doublet_score` and `predicted_doublet` to adata.obs
```
 
> **Alternatives:** [DoubletDetection](https://github.com/JonathanShor/DoubletDetection), [SOLO (scvi-tools)](https://docs.scvi-tools.org/en/stable/user_guide/models/solo.html)
 
---
 
### Step 4 — Normalization
 
Normalize to median count depth, then log1p-transform.
 
```python
adata.layers["counts"] = adata.X.copy()  # Save raw counts
 
sc.pp.normalize_total(adata)   # Median count depth normalization
sc.pp.log1p(adata)             # Log1p transformation (log1PF)
```
 
---
 
### Step 5 — Feature Selection
 
Select the top 2,000 highly variable genes per batch for downstream analysis.
 
```python
sc.pp.highly_variable_genes(adata, n_top_genes=2000, batch_key="sample")
sc.pl.highly_variable_genes(adata)
```
 
---
 
### Step 6 — Dimensionality Reduction
 
Run PCA to capture the main axes of variation.
 
```python
sc.tl.pca(adata)
sc.pl.pca_variance_ratio(adata, n_pcs=50, log=True)  # Inspect elbow to choose # of PCs
```
 
---
 
### Step 7 — Visualization (UMAP)
 
Build a neighborhood graph and embed in 2D with UMAP.
 
```python
sc.pp.neighbors(adata)
sc.tl.umap(adata)
sc.pl.umap(adata, color="sample")
```
 
---
 
### Step 8 — Clustering (Leiden)
 
Cluster cells using the Leiden graph-clustering algorithm at multiple resolutions.
 
```python
sc.tl.leiden(adata, key_added="leiden_res0_02", resolution=0.02)
sc.tl.leiden(adata, key_added="leiden_res0_5",  resolution=0.5)
sc.tl.leiden(adata, key_added="leiden_res2",    resolution=2)
 
sc.pl.umap(adata, color=["leiden_res0_02", "leiden_res0_5", "leiden_res2"], legend_loc="on data")
```
 
---
 
### Step 9 — Re-assess QC & Filter Doublets
 
Visualize doublet scores on UMAP, then remove predicted doublets.
 
```python
adata.obs["predicted_doublet"] = adata.obs["predicted_doublet"].astype("category")
sc.pl.umap(adata, color=["leiden", "predicted_doublet", "doublet_score"], wspace=0.5)
 
adata = adata[~adata.obs["predicted_doublet"].to_numpy()].copy()
```
 
---
 
### Step 10 — Cell-Type Annotation
 
Three complementary approaches:
 
#### A) Marker Gene Dotplot
 
```python
marker_genes = {
    "CD14+ Mono": ["FCN1", "CD14"],
    "CD16+ Mono": ["TCF7L2", "FCGR3A", "LYN"],
    "NK":         ["GNLY", "NKG7", "CD247"],
    "CD4+ T":     ["CD4", "IL7R", "TRBC2"],
    "CD8+ T":     ["CD8A", "CD8B", "GZMK"],
    "B cells":    ["MS4A1", "ITGB1", "PAX5"],
    "Plasma":     ["MZB1", "HSP90B1", "JCHAIN"],
    # ... (see notebook for full list)
}
sc.pl.dotplot(adata, marker_genes, groupby="leiden_res0_5")
```
 
#### B) Automatic Annotation — CellTypist
 
```python
import celltypist as ct
 
ct.models.download_models(model=["Immune_All_Low.pkl"], force_update=True)
predictions = ct.annotate(adata, model="Immune_All_Low.pkl",
                          majority_voting=True, over_clustering="leiden_res0_5")
adata = predictions.to_adata()
sc.pl.umap(adata, color="majority_voting")
```
 
#### C) Enrichment-Based Annotation — decoupler + PanglaoDB
 
```python
import decoupler as dc
 
markers = dc.get_resource("PanglaoDB")
# Filter for human canonical markers
dc.run_mlm(adata, net=markers.rename(columns=dict(cell_type="source", genesymbol="target")),
           weight=None, use_raw=False)
 
# Assign cluster labels by max enrichment score
acts = dc.get_acts(adata, obsm_key="mlm_estimate")
sc.pl.umap(acts, color=["majority_voting", "B cells", "T cells", "Monocytes", "NK cells"],
           wspace=0.5, ncols=3, cmap="RdBu_r", vmin=-2, vmax=2)
```
 
---
 
### Step 11 — Differential Expression
 
Find cluster-specific marker genes using Wilcoxon/t-test.
 
```python
sc.tl.rank_genes_groups(adata, groupby="leiden_res0_5")
sc.tl.filter_rank_genes_groups(adata, min_fold_change=1.5)
sc.pl.rank_genes_groups_dotplot(adata, groupby="leiden_res0_5", standard_scale="var", n_genes=5)
```
 
---
 
## 📚 References
 
- Luecken et al. (2021) — NeurIPS 2021 benchmarking dataset
- McCarthy et al. (2017) — scater QC metrics
- Wolock et al. (2019) — Scrublet doublet detection
- Traag et al. (2019) — Leiden clustering algorithm
- Domínguez Conde et al. (2022) — CellTypist immune atlas
- [Single Cell Best Practices Book](https://www.sc-best-practices.org/)
---
 
## 🔗 Resources
 
- [Scanpy Documentation](https://scanpy.readthedocs.io/)
- [AnnData Documentation](https://anndata.readthedocs.io/)
- [CellTypist](https://github.com/Teichlab/celltypist)
- [decoupler-py](https://github.com/saezlab/decoupler-py)
- [PanglaoDB](https://panglaodb.se/)
- [Single Cell Best Practices](https://www.sc-best-practices.org/)







