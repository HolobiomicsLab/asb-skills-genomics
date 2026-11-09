---
name: jaccard-similarity-assessment-of-de-reproducibility
description: Use when you must quantify robustness/reproducibility of differential
  expression or isoform usage calls across unbalanced single-cell groups, by resampling
  equal cell numbers per group and comparing significant sets across replicates with
  Jaccard indices.
when_to_use_negative:
- The input is a bulk RNA-seq or gene-level count table with naturally balanced replicate
  design, so random downsampling of single cells is unnecessary
- You need a single deterministic DE result rather than a resampling-based variance
  estimate; one DE run provides no basis for cross-round Jaccard computation
- The isoform expression matrix lacks the ZinBWaVE weights required by acorde's DIU
  workflow, so zero-inflation adjustment cannot be performed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package providing the multi-group Differential Isoform Usage (DIU) workflow
    that wraps DESeq2 and edgeR for isoform-level DE testing
  repo: https://github.com/ConesaLab/acorde
- name: DESeq2
  role: One of two DIU testing methods run on each ZinBWaVE-weighted downsampled dataset
- name: edgeR
  role: Second DIU testing method run on each downsampled dataset; results are unioned
    with DESeq2 calls
- name: ZinBWaVE
  role: Computes zero-inflated weights for each downsampled single-cell count matrix
    before DIU testing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:4c7a9fa12f3c8289502d0047b82c67bf54e08f5ce2808b9f9f51b8afa372c0aa
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Jaccard similarity assessment of DE reproducibility

## Summary

A resampling-based benchmarking skill that quantifies how reproducible multi-group differential isoform usage (DIU) calls are under random cell downsampling, by running DESeq2 and edgeR over repeated draws and summarizing detection counts and within-method Jaccard indices across rounds.

## When to use

Use when you must quantify robustness/reproducibility of differential expression or isoform usage calls across unbalanced single-cell groups, by resampling equal cell numbers per group and comparing significant sets across replicates with Jaccard indices.

## When NOT to use

- The input is a bulk RNA-seq or gene-level count table with naturally balanced replicate design, so random downsampling of single cells is unnecessary
- You need a single deterministic DE result rather than a resampling-based variance estimate; one DE run provides no basis for cross-round Jaccard computation
- The isoform expression matrix lacks the ZinBWaVE weights required by acorde's DIU workflow, so zero-inflation adjustment cannot be performed

## Inputs

- Neural-cell isoform-level count expression matrix from the Tasic et al. mouse primary visual cortex Smart-Seq2 scRNA-seq dataset
- Cell type/state identity labels per cell
- Isoform-to-gene relationships for multi-group DIU testing

## Outputs

- downsampling_robustness_metrics.csv with per-round and averaged DE isoform counts and Jaccard indices for DESeq2, edgeR, and the union of methods
- Mean ± SD DE isoform counts and mean ± SD Jaccard indices per method (e.g., DESeq2: 6908 ± 101, J = 0.84 ± 0.02)

## How to apply

Randomly sample an equal number of cells per group (here 45 cells per neural type) from the neural-cell isoform expression matrix, repeating the draw many times (50 rounds) to obtain balanced count matrices. For each round, compute ZinBWaVE zero-inflated weights on the downsampled matrix, then run multi-group DIU testing with both DESeq2 and edgeR via acorde's DIU workflow in R. Record the number of DE isoforms per method per round and compute the within-method Jaccard index between the DE isoform sets across rounds; also compute the union-of-methods set (isoforms significant in either DESeq2 or edgeR) and its Jaccard index. Average counts and Jaccard indices over the 50 rounds and report mean ± standard deviation, writing the summary to downsampling_robustness_metrics.csv for comparison against published benchmarks.

## Related tools

- **acorde** (R package providing the multi-group Differential Isoform Usage (DIU) workflow that wraps DESeq2 and edgeR for isoform-level DE testing) — https://github.com/ConesaLab/acorde
- **DESeq2** (One of two DIU testing methods run on each ZinBWaVE-weighted downsampled dataset)
- **edgeR** (Second DIU testing method run on each downsampled dataset; results are unioned with DESeq2 calls)
- **ZinBWaVE** (Computes zero-inflated weights for each downsampled single-cell count matrix before DIU testing)

## Examples

```
devtools::install_github("ConesaLab/acorde")
```

## Evaluation signals

- Reproduced DESeq2 benchmark: mean 6908 ± 101 DE isoforms with mean Jaccard index 0.84 ± 0.02 across 50 downsampling rounds
- Reproduced edgeR benchmark: mean 6016 ± 410 DE isoforms with mean Jaccard index 0.74 ± 0.02
- Union-of-methods benchmark: 9399 ± 248 DE isoforms with Jaccard index 0.82 ± 0.01
- All computed Jaccard indices fall within [0, 1] and the summary table downsampling_robustness_metrics.csv contains 50 rows (one per round) plus averaged summaries
- Per-round DE counts show method-specific variability consistent with reported SDs (DESeq2 tighter than edgeR)

## Limitations

- Results are specific to the balanced downsampling regime (45 cells per neural type, 50 rounds) and may not transfer to other group sizes or cell type compositions
- The reproducibility metrics measure statistical robustness, not biological validity; DE reproducibility across resamples does not guarantee correct calls
- acorde does not include the long-read transcriptome building or functional analysis steps, so those must be performed with external tools before/after DIU testing

## Evidence

- [results] Across the 50 downsampling runs, DESeq2 detected a mean of 6908 ± 101 DE isoforms with mean Jaccard index 0.84 ± 0.02, while edgeR detected a mean of 6016 ± 410 DE isoforms with mean Jaccard index 0.74 ± 0.02.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow steps of random sampling and weighting.: [evidence span withheld — anchor status: not_found]
- [readme] acorde provides DIU functions on isoform-level matrices.: "acorde contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input."
- [readme] Example dataset availability for reuse.: [evidence span withheld — anchor status: external_doc]
- [intro] Statistical DIU implementations in acorde.: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell"
