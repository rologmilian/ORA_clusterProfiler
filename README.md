# Overrepresentation Analysis with clusterProfiler

A hands-on bioinformatics workshop demonstrating how to perform Overrepresentation Analysis (ORA)
using the clusterProfiler R package. This workshop uses publicly available Alzheimer's disease
RNA-seq data (GSE125583) to identify enriched biological pathways in differentially expressed genes.

---

## Workshop Objectives

- Understand the principles of Overrepresentation Analysis (ORA)
- Filter and prepare differentially expressed gene (DEG) lists for pathway analysis
- Construct an appropriate background gene list
- Use the MSigDB Reactome gene sets (C2 collection) via the msigdbr package
- Run ORA using the `enricher()` function from clusterProfiler
- Generate and interpret multiple enrichment visualizations
- Save and document results reproducibly

---

## Requirements

### R Version

- R 4.0 or higher

### Required CRAN Packages

- tidyverse
- readxl
- circlize
- pheatmap
- RColorBrewer

### Required Bioconductor Packages

- clusterProfiler
- enrichplot
- ggupset
- msigdbr

### Installation Instructions

```r
# Install BiocManager if not already installed
if (!requireNamespace("BiocManager", quietly = TRUE)) {
    install.packages("BiocManager")
}

# Define required packages
cran_packages <- c("tidyverse", "readxl", "circlize", "pheatmap", "RColorBrewer")
bioc_packages <- c("clusterProfiler", "enrichplot", "ggupset", "msigdbr")

# Install missing CRAN packages
install.packages(setdiff(cran_packages, rownames(installed.packages())))

# Install missing Bioconductor packages
BiocManager::install(setdiff(bioc_packages, rownames(installed.packages())))
```

---

## Repository Structure

```
.
|-- metadata_GSE125583.csv
|-- ora_clusterProfiler.qmd
|-- ORA.Rproj
|-- female_DEG_GSE125583.top.table.xlsx
|-- GSE125583_GeneLevel_Normalized_data.csv
|-- male_DEG_GSE125583.top.table.xlsx
```

| File | Description |
|------|-------------|
| `metadata_GSE125583.csv` | Sample metadata for the GSE125583 dataset |
| `ora_clusterProfiler.qmd` | Main Quarto workshop document with all analysis code |
| `ORA.Rproj` | RStudio project file |
| `female_DEG_GSE125583.top.table.xlsx` | DEG results for female Alzheimer's vs female control |
| `GSE125583_GeneLevel_Normalized_data.csv` | Gene-level normalized expression data |
| `male_DEG_GSE125583.top.table.xlsx` | DEG results for male Alzheimer's vs male control |

---

## Input Data

This workshop uses data from the GEO dataset **GSE125583**, which contains RNA-seq data from
human postmortem brain tissue comparing Alzheimer's disease (AD) patients to healthy controls.

Two separate differential expression analyses are provided:

- **Male**: Alzheimer's disease vs. control (male samples)
- **Female**: Alzheimer's disease vs. control (female samples)

DEG results include log2 fold change, p-values, and adjusted p-values (padj) for each gene.

---

## Workshop Workflow

### 1. Loading and Filtering DEGs

- Load DEG results from `.xlsx` files using `readxl`
- Filter significant DEGs using thresholds:
  - |log2FC| > 0.5
  - padj < 0.05
- Separate upregulated and downregulated gene lists as needed

### 2. Preparing the Background Gene List

- Extract all detected gene identifiers to serve as the statistical background for ORA

### 3. Using MSigDB Reactome Gene Sets (C2 Collection)

- Use the `msigdbr` package to retrieve human gene sets from the MSigDB collections
- Format gene sets into a data frame compatible with `enricher()`

### 4. Running ORA with enricher()

- Run `enricher()` from clusterProfiler using:
  - The filtered DEG list as the query
  - The full detected gene list as the background
  - The Reactome C2 MSigDB gene sets as the reference
- Adjust p-values using the Benjamini-Hochberg (BH) method

### 5. Visualizations

The following plots are generated to explore and communicate enrichment results:

| Plot | Function | Description |
|------|----------|-------------|
| Barplot | `barplot()` | Top enriched pathways ranked by gene count or p-value |
| Dotplot | `dotplot()` | Enrichment significance and gene ratio across pathways |
| Heatplot | `heatplot()` | Gene-to-pathway mapping as a heatmap |
| Enrichment Map | `emapplot()` | Network of related enriched pathways |
| Upset Plot | `upsetplot()` | Gene set overlaps across enriched terms |
| Treeplot | `treeplot()` | Hierarchical clustering of enriched pathways |
| Chord Diagram | `circlize` | Relationships between genes and enriched pathways |

---

## Output Files

Running the workshop analysis will generate an `output/` folder containing:

- Filtered DEG tables (`.csv` or `.xlsx`)
- ORA result tables with enriched pathways and statistics
- All visualization plots saved as `.png` or `.pdf` files
- A session info log documenting the R environment and package versions

---

## How to Get Started

1. Clone or download this repository:
   ```bash
   git clone https://github.com/your-username/ora-clusterprofiler-workshop.git
   ```

2. Open the RStudio project by double-clicking `ORA.Rproj`

3. Install all required packages using the installation instructions above

4. Open `ora_clusterProfiler.qmd` in RStudio

5. Run the document chunk by chunk, or render the full Quarto document:
   ```r
   quarto::quarto_render("ora_clusterProfiler.qmd")
   ```

---

## Session Info

At the end of the workshop document, session information is automatically saved to the `output/`
folder as a text file. This documents the exact versions of R and all loaded packages used
during the analysis, ensuring reproducibility and aiding troubleshooting.

---

## License

This workshop material is intended for educational purposes. Please cite the original GSE125583
dataset appropriately if using this data in your own research.
