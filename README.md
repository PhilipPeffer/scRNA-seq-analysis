# Hair follicle single-cell RNA-seq analysis 
In this project, I analyzed a small scRNA-seq dataset consisting of RNA abundances from ~12k hair follicle cells gathered using the 10x 3’ v2 library preparation protocol from 2 different samples, 
as described in [this paper](https://www.jidonline.org/article/S0022-202X(19)33295-6/fulltext) by Takahashi et al.[^1] The study also included cell libraries from 3 additional samples (~3k cells each) that were gathered using the Drop-seq protocol. 
However, these samples were excluded from my analysis for simplicity, as a separate raw read processing pipeline would have been required for the Drop-seq reads. 
The full dataset is available from the Human Cell Atlas database [here](https://explore.data.humancellatlas.org/projects/8b9cb6ae-6a43-4e47-b9fb-3df7aeec941f). 

## Raw read processing
Raw read processing was performed using the Nextflow nf-core scrnaseq pipeline. Nextflow was chosen for its transparency of commands executed and software versions utilized (reproducibility), 
automatic parallelization of processes, as well as its ability to resume execution from cached intermediate result files if a step fails for any reason. 

nf-core pipeline: 
- **Raw data QC** - FastQC 
- **Alignment/Quantification** - several options including Salmon Alevin-fry, STARsolo, and Cell Ranger. I chose to use Salmon Alevin-fry for its strong balance between accuracy and speed,
  and stable performance across datasets[^2]
- **Background/Empty droplet removal** - CellBender 
- **QC Report** - MultiQC

## Downstream Analysis

Downstream analysis was performed following the [“Single-cell best practices” book](https://www.sc-best-practices.org/preamble.html)[^3], primarily using the python-based scverse framework (e.g. scanpy) with Jupyter notebooks. 
However, R-based bioconductor packages and Seurat were utilized on occasion, by converting between scverse anndata and R SingleCellExperiment objects using anndata2ri and the rpy2 library 
to interactively work with R in IPython.

Analysis was organized into the following steps, with a separate notebook for each step:

1. **Quality Control** - filtered out low quality cells using scanpy (python), corrected for ambient RNA using SoupX (R), and removed doublets using scDblFinder (R)
2. **Normalization** - performed both shifted logarithm normalization using scanpy (python) and size factor estimation normalization using scran (R)
3. **Feature Selection** - calculated the binomial deviance for each gene across cells using scry (R) and selected the top 4,000 most highly deviant genes
4. **Dimensionality Reduction** - PCA, t-SNE, and UMAP using scanpy (python)
5. **Clustering** - using scanpy (python), calculated a k nearest neighbors graph on the top 30 principal components from UMAP embedding, and clustered using the Leiden algorithm, varying the resolution parameter
6. **Annotation**
    - Performed manual annotation using scanpy (python) to identify differentially expressed marker genes for each cluster, which were then compared to the marker genes for each cell type identified by
      Takahashi et al.[^1] using overlap analysis
    - Performed automated annotation using CellTypist and scvi-tools CellAssign models (python)

[^1]: Takahashi, R., Grzenda, A., Allison, T. F., Rawnsley, J., Balin, S. J., Sabri, S., Plath, K., & Lowry, W. E. (2020). Defining transcriptional signatures of human hair follicle cell states.
   Journal of Investigative Dermatology, 140(4), 764–773. https://doi.org/10.1016/j.jid.2019.07.726
[^2]: Brüning, R. S., Tombor, L., Schulz, M. H., Dimmeler, S., & John, D. (2022). Comparative analysis of common alignment tools for single-cell RNA sequencing. GigaScience, 11. https://doi.org/10.1093/gigascience/giac001
[^3]: Heumos, L., Schaar, A.C., Lance, C. et al. Best practices for single-cell analysis across modalities. Nat Rev Genet (2023). https://doi.org/10.1038/s41576-023-00586-w
