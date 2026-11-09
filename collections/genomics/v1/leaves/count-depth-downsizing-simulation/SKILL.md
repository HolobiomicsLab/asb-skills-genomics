---
name: count-depth-downsizing-simulation
description: Use when you have a raw single-cell count matrix with an independent
  gold-standard DE gene list and want to compare normalization methods under controlled,
  simulated loss of sequencing depth/dropout rather than only on undegraded data.
when_to_use_negative:
- Your aim is to test library-size or capture-efficiency confounders other than count-depth
  loss — the Uniform(0.2,1) multiplication specifically simulates depth/dropout reduction
- You have no independent gold-standard DE gene list, so top-N recovery against validated
  genes cannot be computed
- Input data are already normalized (e.g., log1p-transformed) — downsizing must be
  applied to raw counts before normalization
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: scKWARN (LocASN)
  role: Normalization method under evaluation; LocASN computes per-cell scaling factors
    from raw or downsized count matrices (general or sparse input), returning NormalizedData
    and scalingFactor
  repo: https://github.com/cyhsuTN/scKWARN
- name: R
  role: Implementation language for the downsizing simulation, per-gene ranking, and
    top-N proportion computation
- name: devtools
  role: Installs the scKWARN package from GitHub before running LocASN
  repo: https://github.com/cyhsuTN/scKWARN
- name: Matrix
  role: Supports sparse-matrix representation of the count matrix passed to LocASN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:0b20f43ead8894ab8dcbc21032b2a1648b44da6293463ecd670f3b484908f278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cyhsuTN/scKWARN
  tools:
  - Matrix
  - R
  - devtools
  - scKWARN
  - scKWARN (LocASN)
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# Count-depth downsizing simulation

## Summary

Benchmark normalization methods for robustness to count-depth loss (high dropout) by multiplying single-cell counts by factors drawn from Uniform(0.2,1) under none/half/all cell downsizing regimes, then scoring recovery of a gold-standard DE gene set (e.g., the 718 qRT-PCR-validated genes in GSE29087).

## When to use

Use when you have a raw single-cell count matrix with an independent gold-standard DE gene list and want to compare normalization methods under controlled, simulated loss of sequencing depth/dropout rather than only on undegraded data.

## When NOT to use

- Your aim is to test library-size or capture-efficiency confounders other than count-depth loss — the Uniform(0.2,1) multiplication specifically simulates depth/dropout reduction
- You have no independent gold-standard DE gene list, so top-N recovery against validated genes cannot be computed
- Input data are already normalized (e.g., log1p-transformed) — downsizing must be applied to raw counts before normalization

## Inputs

- Raw UMI/count matrix from GEO accession GSE29087 (92 mouse embryonic cells and fibroblasts), subset to 7896 genes
- Gold-standard DE gene set: 718 qRT-PCR-validated genes from the source study
- Downsizing regime specification: factor ~ Uniform(0.2, 1) applied to none, half, or all cells

## Outputs

- Per-cell normalized matrices (NormalizedData = counts/scaling factor) and per-cell scalingFactor vectors for each method x downsizing combination
- Proportion of the 718 gold-standard DE genes among the top N ranked genes for each method and setting
- Three-panel summary table (none/half/all downsized) over all six methods, saved as figure5_topN_proportions.csv

## How to apply

Download the source dataset (GEO GSE29087: 92 mouse embryonic cells and fibroblasts) and subset the matrix to the 7896 genes described in the paper. Apply three downsizing regimes in which counts are multiplied by factors drawn from Uniform(0.2,1) for none, half, or all cells, so the 'all' setting imposes severe dropout. Normalize each downsized matrix with all candidate methods — including scKWARN's LocASN via Result <- LocASN(countmatrix = ...), which accepts general or sparse matrices, and competitors such as sctransform and PsiNorm — extracting NormalizedData (counts/scaling factor) and per-cell scalingFactor from each. Rank genes per cell/setting and compute the proportion of the 718 qRT-PCR-validated gold-standard genes among the top N ranked genes for every method-by-downsizing combination. The rationale is that a method that correctly estimates scaling factors under reduced count depth will keep validated DE genes in top ranks, whereas methods like sctransform whose proportion estimates drop under high dropout degrade.

## Related tools

- **scKWARN (LocASN)** (Normalization method under evaluation; LocASN computes per-cell scaling factors from raw or downsized count matrices (general or sparse input), returning NormalizedData and scalingFactor) — https://github.com/cyhsuTN/scKWARN
- **R** (Implementation language for the downsizing simulation, per-gene ranking, and top-N proportion computation)
- **devtools** (Installs the scKWARN package from GitHub before running LocASN) — https://github.com/cyhsuTN/scKWARN
- **Matrix** (Supports sparse-matrix representation of the count matrix passed to LocASN)

## Examples

```
library(devtools); install_github("cyhsuTN/scKWARN"); Result <- LocASN(countmatrix = as(downsampled_counts, "sparseMatrix"))
```

## Evaluation signals

- figure5_topN_proportions.csv exists with a three-panel structure: none/half/all downsizing settings by all six normalization methods
- In the no- and half-downsizing panels, sctransform attains the highest gold-standard top-N proportions, matching Figure 5 of the paper
- In the all-cells-downsized panel, scKWARN (LocASN) and PsiNorm lead while sctransform's gold-standard proportion drops significantly, matching the reported high-dropout finding
- LocASN returns both NormalizedData (counts/scaling factor, a dgCMatrix) and a per-cell scalingFactor vector, confirming normalization ran on each downsized matrix
- Each downsized matrix's counts are bounded below the originals, with every multiplied cell factor drawn within Uniform(0.2, 1)

## Limitations

- The severity regime is fixed to factors drawn from Uniform(0.2, 1); other dropout depths or dropout distributions are not covered
- Validation of method superiority depends on the specific 718 qRT-PCR-validated gene set of GSE29087; conclusions may not transfer to datasets without such a gold standard
- scKWARN requires raw count input (general or sparse matrix); it operates on counts and its scaling factors are not interpretable if passed log-transformed data

## Evidence

- [methods] LocASN called on a count matrix accepts general or sparse input: "### Inputs: countmatrix (general matrix or sparse matrix)"
- [methods] LocASN outputs NormalizedData and per-cell scalingFactor: "Outputs: NormalizedData (normalized data = counts/scaling factor) and scalingFactor (a scaling factor for each cell)"
- [abstract] scKWARN corrects technical confounders without distribution or count-depth assumptions, motivating its use under simulated dropout: "We present scKWARN, a Kernel Weighted Average Robust Normalization designed to correct known or hidden technical confounders without assuming specific data distributions or count-depth relationships."
- [readme] README shows the concrete LocASN invocation on a simulated count matrix: "Result <- LocASN(countmatrix = countsimdata)
#Result <- LocASN(countmatrix = as(countsimdata,"sparseMatrix"))"
