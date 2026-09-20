---
name: single-cell-isoform-downsampling-robustness-testing
description: Use when you must assess how stable isoform-level DE/DIU calls are to
  random cell sampling in single-cell isoform count matrices, e.g. before trusting
  multi-group DESeq2/edgeR DIU results on Tasic-style neural Smart-Seq2 data.
when_to_use_negative:
- Input is a gene-level (not isoform-level) expression matrix — acorde's DIU testing
  requires isoform quantification.
- The goal is bulk RNA-seq differential expression where per-cell sampling variability
  and zero inflation are not relevant, so downsampling robustness estimation is unnecessary.
- You need long-read transcriptome building or functional annotation of isoforms —
  the acorde R package explicitly excludes these external steps.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package providing the multi-group DIU workflow applied to each downsampled,
    ZinBWaVE-weighted count matrix with DESeq2 and edgeR
  repo: https://github.com/ConesaLab/acorde
- name: DESeq2
  role: 'One of the two DIU testing methods whose per-round DE isoform counts and
    Jaccard indices are benchmarked (target: 6908±101, J=0.84±0.02)'
- name: edgeR
  role: 'Second DIU testing method benchmarked alongside DESeq2 (target: 6016±410,
    J=0.74±0.02)'
- name: ZinBWaVE
  role: Computes zero-inflated weights for each downsampled isoform count matrix before
    DIU testing
- name: R
  role: Runtime environment in which the downsampling, weighting, DIU testing, and
    Jaccard summarization are executed
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:a34a372b68ddddd912103454064194902e39d93498751d2f84c977da6d87b096
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-isoform-downsampling-robustness-testing

## Summary

A benchmarking workflow that quantifies the reproducibility of multi-group differential isoform usage (DIU) calls by repeating random downsampling of scRNA-seq cells, running DESeq2 and edgeR on each replicate, and summarizing per-method DE counts and within-method Jaccard indices against published benchmarks.

## When to use

Use when you must assess how stable isoform-level DE/DIU calls are to random cell sampling in single-cell isoform count matrices, e.g. before trusting multi-group DESeq2/edgeR DIU results on Tasic-style neural Smart-Seq2 data.

## When NOT to use

- Input is a gene-level (not isoform-level) expression matrix — acorde's DIU testing requires isoform quantification.
- The goal is bulk RNA-seq differential expression where per-cell sampling variability and zero inflation are not relevant, so downsampling robustness estimation is unnecessary.
- You need long-read transcriptome building or functional annotation of isoforms — the acorde R package explicitly excludes these external steps.

## Inputs

- isoform-level single-cell count matrix (Tasic et al. mouse primary visual cortex Smart-Seq2 neural cells)
- cell-level identity labels (neural cell type annotations)
- isoform-to-gene relationships
- target cells-per-type value for downsampling (n = 45) and number of random rounds (50)

## Outputs

- downsampling_robustness_metrics.csv with mean ± SD DE isoform counts and Jaccard indices per method (DESeq2, edgeR) and for the union of methods
- per-round DE isoform sets for each downsampling replicate

## How to apply

Randomly sample a fixed number of cells per cell type (here 45 neural cells per type) from the isoform-level count matrix, repeating the draw 50 times. For each downsampled matrix, compute ZinBWaVE zero-inflated weights, then run multi-group DIU testing with DESeq2 and separately with edgeR via acorde's DIU workflow in R. Per round, record the number of DE isoforms per method and compute the within-method Jaccard index between DE isoform sets across rounds; also compute the union-of-methods DE count and Jaccard index (isoforms significant in either DESeq2 or edgeR). Average counts and Jaccard indices over the 50 rounds and write the summary to downsampling_robustness_metrics.csv, checking reproduction against the benchmarks: DESeq2 6908±101 DE isoforms with J=0.84±0.02, edgeR 6016±410 with J=0.74±0.02, union 9399±248 with J=0.82±0.01.

## Related tools

- **acorde** (R package providing the multi-group DIU workflow applied to each downsampled, ZinBWaVE-weighted count matrix with DESeq2 and edgeR) — https://github.com/ConesaLab/acorde
- **DESeq2** (One of the two DIU testing methods whose per-round DE isoform counts and Jaccard indices are benchmarked (target: 6908±101, J=0.84±0.02))
- **edgeR** (Second DIU testing method benchmarked alongside DESeq2 (target: 6016±410, J=0.74±0.02))
- **ZinBWaVE** (Computes zero-inflated weights for each downsampled isoform count matrix before DIU testing)
- **R** (Runtime environment in which the downsampling, weighting, DIU testing, and Jaccard summarization are executed) — https://github.com/ConesaLab/acorde

## Examples

```
devtools::install_github("ConesaLab/acorde")
```

## Evaluation signals

- Reproduced DESeq2 mean DE isoform count matches 6908 ± 101 with mean Jaccard 0.84 ± 0.02
- Reproduced edgeR mean DE isoform count matches 6016 ± 410 with mean Jaccard 0.74 ± 0.02
- Union-of-methods DE isoform count matches 9399 ± 248 with Jaccard 0.82 ± 0.01
- Exactly 50 downsampling rounds with 45 cells sampled per neural type per round are completed
- Summary table downsampling_robustness_metrics.csv is produced with per-method and union metrics averaged over all rounds

## Limitations

- Jaccard/robustness metrics are specific to the Tasic neural Smart-Seq2 isoform matrix and the n=45 cells-per-type design; other datasets or sample sizes will not match the published benchmarks.
- Results depend on ZinBWaVE zero-inflated weighting; skipping the weighting step breaks comparability with the reported values.
- Functional insights and downstream interpretation from the acorde pipeline require further experimental validation, per the authors.

## Evidence

- [intro] We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses"
- [readme] acorde provides DIU from isoform-level matrices: "*acorde* contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input"
- [readme] Manuscript isoform expression matrix shipped as package data: [evidence span withheld — anchor status: external_doc]
- [intro] Tasic Smart-Seq2 dataset used in study: "scRNA-seq Smart-Seq2 (primary visual cortex, generated by Tasic et al.43,44)"
