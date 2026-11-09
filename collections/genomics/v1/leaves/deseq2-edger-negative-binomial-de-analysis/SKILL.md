---
name: deseq2-edger-negative-binomial-de-analysis
description: Use when multi-group differential expression/isoform usage must be tested
  on single-cell isoform-level count matrices across cell types, and reproducibility
  of DE calls under cell-number balancing (downsampling) must be quantified with DESeq2
  and edgeR.
when_to_use_negative:
- The goal is co-expression network construction (percentile correlations, isoform
  clustering) rather than DIU/DE significance testing
- Long-read transcriptome building or functional annotation of isoforms is needed
  — acorde explicitly excludes these external steps
- Data are bulk RNA-seq with balanced biological replicates, where zero-inflation
  weighting of single-cell dropout is unnecessary
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: acorde
  role: R package providing the multi-group DIU workflow that wraps DESeq2/edgeR testing
    on single-cell isoform expression matrices
  repo: https://github.com/ConesaLab/acorde
- name: DESeq2
  role: Negative-binomial model used within acorde's DIU workflow for multi-group
    differential isoform usage testing on ZinBWaVE-weighted downsampled counts
- name: edgeR
  role: Alternative negative-binomial method run in parallel with DESeq2 to obtain
    union-of-methods DIU calls and assess concordance
- name: ZinBWaVE
  role: Computes zero-inflated weights applied to each downsampled count matrix before
    DIU testing
- name: R
  role: Language in which the downsampling, weighting, DIU testing, and Jaccard computation
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:8858cbf09b789d579abc50a2299af400fed1ad232de7a447950ffb09011f3061
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - DESeq2
  - R
  - ZinBWaVE
  - acorde
  - edgeR
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# deseq2-edger-negative-binomial-de-analysis

## Summary

Applies DESeq2 and edgeR negative-binomial testing, coupled with ZinBWaVE zero-inflated weights, to perform multi-group Differential Isoform Usage (DIU) analysis on single-cell isoform count matrices via acorde's DIU workflow, and assesses robustness by downsampling cells across repeated rounds and scoring reproducibility with Jaccard indices.

## When to use

Use when multi-group differential expression/isoform usage must be tested on single-cell isoform-level count matrices across cell types, and reproducibility of DE calls under cell-number balancing (downsampling) must be quantified with DESeq2 and edgeR.

## When NOT to use

- The goal is co-expression network construction (percentile correlations, isoform clustering) rather than DIU/DE significance testing
- Long-read transcriptome building or functional annotation of isoforms is needed — acorde explicitly excludes these external steps
- Data are bulk RNA-seq with balanced biological replicates, where zero-inflation weighting of single-cell dropout is unnecessary

## Inputs

- Neural-cell isoform-level expression/quantification matrix (single-cell counts, e.g. Smart-Seq2 Tasic et al. mouse primary visual cortex data)
- Isoform-to-gene relationships and cell-level identity labels (cell type or state)
- Number of cells per group to sample per downsampling round and number of random rounds (e.g. n=45, 50 rounds)

## Outputs

- Per-round DE isoform counts for DESeq2 and edgeR, with mean ± SD across rounds
- Within-method Jaccard indices between DE isoform sets across rounds (mean ± SD)
- Union-of-methods DE isoform count and Jaccard index
- Summary table (e.g. downsampling_robustness_metrics.csv)

## How to apply

Balance sample sizes by randomly sampling an equal number of cells per neural type (e.g. 45 cells) from the neural-cell isoform expression matrix, repeating the random draw many times (e.g. 50 rounds). For each downsampled count matrix, compute ZinBWaVE zero-inflated weights and run acorde's multi-group DIU testing separately with DESeq2 and with edgeR in R. Record the number of DE isoforms detected per method per round and compute within-method Jaccard indices between DE isoform sets across rounds; also compute the union-of-methods DE set (isoforms significant in either tool). Average counts and Jaccard indices over rounds to report method-specific stability; benchmark values in the source study were DESeq2: 6908±101 DE isoforms, J=0.84±0.02; edgeR: 6016±410, J=0.74±0.02; union: 9399±248, J=0.82±0.01.

## Related tools

- **acorde** (R package providing the multi-group DIU workflow that wraps DESeq2/edgeR testing on single-cell isoform expression matrices) — https://github.com/ConesaLab/acorde
- **DESeq2** (Negative-binomial model used within acorde's DIU workflow for multi-group differential isoform usage testing on ZinBWaVE-weighted downsampled counts)
- **edgeR** (Alternative negative-binomial method run in parallel with DESeq2 to obtain union-of-methods DIU calls and assess concordance)
- **ZinBWaVE** (Computes zero-inflated weights applied to each downsampled count matrix before DIU testing)
- **R** (Language in which the downsampling, weighting, DIU testing, and Jaccard computation are executed)

## Examples

```
devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE)
```

## Evaluation signals

- Reproduced DESeq2 mean DE isoform count falls near 6908 ± 101 with mean Jaccard index 0.84 ± 0.02
- Reproduced edgeR mean DE isoform count falls near 6016 ± 410 with mean Jaccard index 0.74 ± 0.02
- Union-of-methods DE isoform count and Jaccard match benchmarks: 9399 ± 248 and 0.82 ± 0.01
- Each downsampled group contains exactly the target number of cells (e.g. 45) per neural type in every round
- Summary table (downsampling_robustness_metrics.csv) contains per-round and averaged counts and Jaccard indices for both methods and the union

## Limitations

- DESeq2 and edgeR disagreed in sensitivity: edgeR detected fewer DE isoforms (6016 vs 6908) with lower across-round reproducibility (J=0.74 vs 0.84), so single-method calls may be unstable
- acorde does not include long-read transcriptome building or functional analysis; these require external tools and pre-annotated references (e.g. IsoAnnotLite v2.6 with tappAS mouse RefSeq 78)
- Functional annotation of long read-defined isoforms depends on a compatible reference transcriptome; de novo annotation tooling was still a work in progress

## Evidence

- [readme] acorde contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input.: "acorde contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input."
- [intro] We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell"
- [readme] The isoform expression matrix employed during the study is provided as internal data in the package.: [evidence span withheld — anchor status: external_doc]
