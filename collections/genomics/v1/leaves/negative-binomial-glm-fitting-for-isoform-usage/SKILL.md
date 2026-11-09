---
name: negative-binomial-glm-fitting-for-isoform-usage
description: Use when an isoform-level scRNA-seq quantification matrix with cell-type
  labels is available and the goal is to detect genes whose isoform usage varies across
  cell types (DIU) or whose same-gene isoforms switch co-expressed cluster membership
  (coDIU).
when_to_use_negative:
- Input is a gene-level expression matrix rather than isoform-level quantification
  — the GLM tests isoform usage and requires transcript-resolution counts.
- Data are bulk RNA-seq without per-cell type/state labels — the cluster*cell-type
  and gene*cell-type interaction terms cannot be parameterized.
- Goal is long-read transcriptome building or functional annotation itself — the acorde
  package explicitly excludes these steps, deferring them to SQANTI3/IsoAnnotLite
  and tappAS.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: acorde (R package)
  role: Implements the DIU/coDIU negative-binomial GLM tests on isoform-level single-cell
    counts and ships the manuscript example dataset as internal package data
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Runtime environment for fitting the negative-binomial GLMs and applying FDR
    correction to the interaction terms
- name: tappAS
  role: Downstream functional readout coupled to the DIU/coDIU gene sets via functionally
    annotated isoform models
  repo: https://github.com/ConesaLab/tappAS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:e6ae2d5bd9ca69a620c19ee6d18a4479d5d0872dc92c43b0a5c3d49a4cc39280
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  - acorde (R package)
  - tappAS
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# negative-binomial-glm-fitting-for-isoform-usage

## Summary

Fits negative-binomial GLMs to isoform-level single-cell counts, modeling isoform usage as a function of cluster, cell type, and their interaction terms, to call DIU and co-differential isoform usage (coDIU) genes as implemented in the acorde R package.

## When to use

Use when an isoform-level scRNA-seq quantification matrix with cell-type labels is available and the goal is to detect genes whose isoform usage varies across cell types (DIU) or whose same-gene isoforms switch co-expressed cluster membership (coDIU).

## When NOT to use

- Input is a gene-level expression matrix rather than isoform-level quantification — the GLM tests isoform usage and requires transcript-resolution counts.
- Data are bulk RNA-seq without per-cell type/state labels — the cluster*cell-type and gene*cell-type interaction terms cannot be parameterized.
- Goal is long-read transcriptome building or functional annotation itself — the acorde package explicitly excludes these steps, deferring them to SQANTI3/IsoAnnotLite and tappAS.

## Inputs

- isoform-level quantification matrix (single-cell, e.g. Tasic et al. mouse visual cortex Smart-Seq2)
- isoform-to-gene relationship map
- cell-level identity labels (cell type or state)
- isoform cluster assignments from the co-expression clustering step (to identify same-gene isoform pairs in shared clusters)

## Outputs

- table of coDIU genes and their co-expressed isoforms (codiu_results.csv)
- list of DIU and co-DIU genes with FDR-corrected GLM interaction term statistics

## How to apply

Starting from isoform co-expression clusters built with percentile correlations, scan the isoform cluster assignments to identify candidate same-gene pairs with at least two isoforms from the same gene present in the same clusters. For each candidate, fit a negative-binomial GLM with cluster, cell type, and their interaction as covariates, testing both the cluster*cell-type and gene*cell-type interaction terms. Apply FDR multiple-testing correction and retain pairs with significant cluster*cell-type interaction (FDR < 0.05) and non-significant gene*cell-type interaction (FDR > 0.05), which flags cell type-dependent isoform co-usage invisible at the gene level. Summarize retained coDIU genes and their co-expressed isoforms (e.g., to codiu_results.csv); the acorde package provides the functions and example data used in the manuscript.

## Related tools

- **acorde (R package)** (Implements the DIU/coDIU negative-binomial GLM tests on isoform-level single-cell counts and ships the manuscript example dataset as internal package data) — https://github.com/ConesaLab/acorde
- **R** (Runtime environment for fitting the negative-binomial GLMs and applying FDR correction to the interaction terms)
- **tappAS** (Downstream functional readout coupled to the DIU/coDIU gene sets via functionally annotated isoform models) — https://github.com/ConesaLab/tappAS

## Examples

```
devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE)
```

## Evaluation signals

- Retained coDIU pairs must pass cluster*cell-type FDR < 0.05 while gene*cell-type FDR > 0.05.
- On the Tasic et al. mouse visual cortex Smart-Seq2 dataset, the analysis should reproduce 1784 coDIU genes involving 5274 co-expressed isoforms.
- Only same-gene isoform pairs (>= 2 isoforms from one gene in a shared cluster) should enter the GLM candidate scan.
- Output table (codiu_results.csv) must map each coDIU gene to its co-expressed isoforms consistently with the isoform cluster assignments.
- Candidate counts must reconcile with the co-expression clustering step's isoform-to-gene mapping.

## Limitations

- Functional interpretation of coDIU outputs requires a functionally annotated transcriptome via isoAnnotLite/pre-annotated references; de novo annotation is not yet available.
- Long-read transcriptome definition and functional analysis are not included in the acorde R package itself.
- Reported biological insights from the coDIU analysis are hypothesis-generating and require further experimental validation.

## Evidence

- [intro] coDIU detection defined and implemented in acorde: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell"
- [abstract] acorde detects coDIU genes across cell types: "By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types"
- [readme] acorde provides DIU/coDIU functions from an isoform expression matrix: "*acorde* contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input."
- [readme] required inputs: quantification matrix, isoform-to-gene map, cell labels: [evidence span withheld — anchor status: external_doc]
- [readme] coDIU analysis coupled to functional readout via tappAS: "Differential and co-Differential Isoform Usage analysis. To couple these analysis with a biologically interpretable readout, we incorporate functional annotations onto isoform models"
- [readme] manuscript isoform expression matrix shipped as internal data: [evidence span withheld — anchor status: external_doc]
