# Overrepresentation Analysis with clusterProfiler

A hands-on bioinformatics workshop demonstrating how to perform Overrepresentation Analysis (ORA)
using the clusterProfiler R package. This workshop uses publicly available Alzheimer's disease
RNA-seq data (GSE125583) to identify enriched biological pathways in differentially expressed genes.

The workshop has two Quarto documents:

| Document | Use it to |
|----------|-----------|
| `ora_clusterProfiler.qmd` | Learn ORA step by step with one molecular signature (GO Biological Process, C5BP) |
| `ora_loop.qmd` | Run the same analysis for several molecular signatures at once |

---

## Workshop Objectives

- Understand the principles of Overrepresentation Analysis (ORA)
- Filter and prepare differentially expressed gene (DEG) lists for pathway analysis
- Construct an appropriate background gene list
- Use MSigDB gene sets via the msigdbr package
- Run ORA using the `enricher()` function from clusterProfiler
- Generate and interpret multiple enrichment visualizations
- Run several molecular signatures in a loop
- Save and document results reproducibly

---

## Requirements

### R Version

- R 4.0 or higher

### Required CRAN Packages

- tidyverse
- circlize
- pheatmap
- RColorBrewer
- here

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
cran_packages <- c("tidyverse", "circlize", "pheatmap", "RColorBrewer", "here")
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
|-- AD_vs_ctrol_female.csv
|-- AD_vs_ctrol_male.csv
|-- ora_clusterProfiler.qmd
|-- ora_loop.qmd
|-- ORA.Rproj
|-- README.md
|-- input/          (created when you run a document; put the DEG file to analyze here)
|-- output/         (created by ora_clusterProfiler.qmd)
|-- output_loop/    (created by ora_loop.qmd)
```

| File | Description |
|------|-------------|
| `AD_vs_ctrol_female.csv` | DESeq2 results for female Alzheimer's vs female control |
| `AD_vs_ctrol_male.csv` | DESeq2 results for male Alzheimer's vs male control |
| `ora_clusterProfiler.qmd` | Workshop document: ORA for a single molecular signature, step by step |
| `ora_loop.qmd` | Standalone document: ORA for several molecular signatures in a loop |
| `ORA.Rproj` | RStudio project file |

---

## Input Data

This workshop uses data from the GEO dataset **GSE125583**, which contains RNA-seq data from
human postmortem brain tissue comparing Alzheimer's disease (AD) patients to healthy controls.

Two separate differential expression analyses are provided:

- **Male**: Alzheimer's disease vs. control (male samples) - `AD_vs_ctrol_male.csv`
- **Female**: Alzheimer's disease vs. control (female samples) - `AD_vs_ctrol_female.csv`

Each file is a DESeq2 results table with one row per gene and the columns `entrez_id`, `symbol`,
`baseMean`, `log2FoldChange`, `lfcSE`, `stat`, `pvalue` and `padj`.

Copy the file you want to analyze into the `input/` folder. Both documents read
`input/AD_vs_ctrol_male.csv` by default; to analyze the female data, change the file name in the
"Upload the results of the DESeq2 analysis" chunk and `name_of_comparison` in the settings.

---

## Workshop Workflow (`ora_clusterProfiler.qmd`)

### 1. Loading and Filtering DEGs

- Load DESeq2 results from a `.csv` file using `readr`
- Filter significant DEGs using thresholds:
  - |log2FC| > 0.5
  - padj < 0.05
- Remove genes without a gene symbol and duplicated symbols
- Separate upregulated and downregulated gene lists as needed

### 2. Preparing the Background Gene List

- Extract all detected gene symbols to serve as the statistical background (universe) for ORA

### 3. Using MSigDB Gene Sets

- Use the `msigdbr` package to retrieve human gene sets from the MSigDB collections
- The workshop uses GO Biological Process (C5, `GO:BP`); `msigdbr_collections()` lists the others
- Format gene sets into a data frame compatible with `enricher()`

### 4. Running ORA with enricher()

- Run `enricher()` from clusterProfiler using:
  - The filtered DEG list as the query
  - The full detected gene list as the background
  - The MSigDB gene sets as the reference
- Adjust p-values using the Benjamini-Hochberg (BH) method
- Filter the significant pathways by keywords for plotting (adapt them to your biological context)

### 5. Visualizations

The following plots are generated to explore and communicate enrichment results:

| Plot | Function | Description |
|------|----------|-------------|
| Barplot | `barplot()` | Top enriched pathways ranked by gene count or q-value |
| Dotplot | `dotplot()` | Enrichment significance and gene ratio across pathways |
| Heatplot | `heatplot()` | Gene-to-pathway mapping, colored by log2 fold change |
| Gene frequency barplot | `ggplot2` | Number of pathways each gene appears in, colored by log2 fold change |
| Enrichment Map | `emapplot()` | Network of related enriched pathways |
| Upset Plot | `upsetplot()` | Gene set overlaps across enriched terms |
| Treeplot | `treeplot()` | Hierarchical clustering of enriched pathways |
| Chord Diagram | `circlize` | Relationships between genes and enriched pathways |

---

## Running Several Molecular Signatures (`ora_loop.qmd`)

`ora_loop.qmd` runs the same Overrepresentation Analysis as `ora_clusterProfiler.qmd`, but for
several molecular signatures (collections) at once. It is standalone: it loads the data and defines
the DEGs itself. The results table and figures for each comparison and collection are saved in
their own folder inside `output_loop/`, named comparison_collection
(e.g. `output_loop/AD_vs_ctrol_male_C2KEGG/`), so they never overwrite the results of
`ora_clusterProfiler.qmd` in `output/`.

The collections to run are listed in a table in the "Define the collections to run" chunk:

| Label | Collection | Subcollection |
|-------|------------|---------------|
| C2KEGG | C2 | `CP:KEGG_MEDICUS` (use `CP:KEGG_LEGACY` for the classic KEGG gene sets) |
| C2Reactome | C2 | `CP:REACTOME` |
| C5BP | C5 | `GO:BP` |
| C7immune | C7 | `IMMUNESIGDB` |

- Add, remove or change rows to run other collections; give each row a different label
- Each row has an optional keyword filter (`NA` means no filter)
- For each collection, the loop saves the results table and 7 figures (barplot, dotplot, heatplot,
  gene frequency barplot, upset plot, enrichment map and treeplot)
- If a collection has no significant pathways, only the (empty) results table is saved
- At the end, a summary table shows the number of significant and plotted pathways per collection

The DEG cutoffs, background, `padj_cutoff` and `min_geneset_size` are the same in both documents.
If you change them in one document, change them in the other too.

---

## Output Files

Each document saves its results in its own folder, with one subfolder per comparison and
molecular signature:

```
output/                                 (ora_clusterProfiler.qmd)
|-- AD_vs_ctrol_male_degs.csv
|-- AD_vs_ctrol_male_C5BP/
|-- clusterProfiler_sessionInfo.txt

output_loop/                            (ora_loop.qmd)
|-- AD_vs_ctrol_male_degs.csv
|-- AD_vs_ctrol_male_C2KEGG/
|-- AD_vs_ctrol_male_C2Reactome/
|-- AD_vs_ctrol_male_C5BP/
|-- AD_vs_ctrol_male_C7immune/
|-- clusterProfiler_loop_sessionInfo.txt
```

- `*_degs.csv`: the filtered DEG table for the comparison
- Each comparison_signature folder contains:
  - `*_resclusterprofiler.csv`: the ORA results table with all significant pathways and statistics
  - The visualization plots as `.pdf` files
- `*_sessionInfo.txt`: a session info log documenting the R environment and package versions

---

## How to Get Started

1. Clone or download this repository:
   ```bash
   git clone https://github.com/your-username/ora-clusterprofiler-workshop.git
   ```

2. Open the RStudio project by double-clicking `ORA.Rproj`

3. Install all required packages using the installation instructions above

4. Copy `AD_vs_ctrol_male.csv` (or the female file) into the `input/` folder

5. Open `ora_clusterProfiler.qmd` in RStudio

6. Run the document chunk by chunk, or render the full Quarto document:
   ```r
   quarto::quarto_render("ora_clusterProfiler.qmd")
   ```

7. Once you are familiar with the single-signature workflow, open `ora_loop.qmd` to run several
   signatures at once

### Troubleshooting

- **"file does not exist" when reading the CSV:** check that the file is in the `input/` folder and
  that R's working directory is the project folder (`getwd()` should end in `ORA`). Open the project
  with `ORA.Rproj`, and check that your `~/.Rprofile` does not contain a `setwd()` line.
- **"object not found":** a chunk was run before the chunks it depends on. Use **Run All Chunks
  Above** (`Cmd+Option+P` on Mac, `Ctrl+Alt+P` on Windows), then run the chunk again.
- **Plots appear under each chunk instead of in the Plots pane:** this is RStudio's default for
  Quarto documents. To change it, click the gear icon next to **Render** and choose
  **Chunk Output in Console**.

---

## Session Info

At the end of each document, session information is automatically saved as a text file
(`output/clusterProfiler_sessionInfo.txt` or `output_loop/clusterProfiler_loop_sessionInfo.txt`).
This documents the exact versions of R and all loaded packages used during the analysis,
ensuring reproducibility and aiding troubleshooting.

---

## License

This workshop material is intended for educational purposes. Please cite the original GSE125583
dataset appropriately if using this data in your own research.
