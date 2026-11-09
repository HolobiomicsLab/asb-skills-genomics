---
name: zinbave-zero-inflation-weighting
description: Use when preparing sparse single-cell isoform count matrices — especially
  after balancing/downsampling cell numbers across groups — for differential isoform
  usage (DIU) testing with DESeq2 or edgeR via acorde.
when_to_use_negative:
- The input is bulk RNA-seq or already aggregated pseudo-bulk counts, where zero inflation
  is not the dominant sparsity mechanism and ZinBWaVE weighting is unnecessary.
- The goal is gene-level differential expression rather than isoform-level differential
  isoform usage (DIU) within the acorde pipeline.
- The count matrix has already been weighted or the analysis plan does not include
  DESeq2/edgeR testing that can consume weights.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ZinBWaVE
  role: computes zero-inflated weights applied to downsampled isoform count matrices
    before DIU testing
- name: acorde
  role: provides the DIU/coDIU workflow that consumes the weighted single-cell isoform-level
    expression matrix
  repo: https://github.com/ConesaLab/acorde
- name: DESeq2
  role: multi-group DIU testing on ZinBWaVE-weighted downsampled datasets (one of
    two methods benchmarked)
- name: edgeR
  role: second multi-group DIU testing method run on the same weighted downsampled
    datasets
- name: R
  role: implementation language for the acorde DIU pipeline and weighting step
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:8d401a36ab9e0d7b5775bdb603de32e06b04f06b0a91864d2f6083a929379af0
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# zinbave-zero-inflation-weighting

## Summary

Applies ZinBWaVE zero-inflated Gaussian-model weights to single-cell isoform-level count matrices (e.g. the Tasic et al. Smart-Seq2 mouse visual cortex dataset) so that DIU testing with DESeq2 or edgeR in the acorde workflow accounts for excess zeros introduced by scRNA-seq sparsity and downsampling.

## When to use

Use when preparing sparse single-cell isoform count matrices — especially after balancing/downsampling cell numbers across groups — for differential isoform usage (DIU) testing with DESeq2 or edgeR via acorde.

## When NOT to use

- The input is bulk RNA-seq or already aggregated pseudo-bulk counts, where zero inflation is not the dominant sparsity mechanism and ZinBWaVE weighting is unnecessary.
- The goal is gene-level differential expression rather than isoform-level differential isoform usage (DIU) within the acorde pipeline.
- The count matrix has already been weighted or the analysis plan does not include DESeq2/edgeR testing that can consume weights.

## Inputs

- isoform-level scRNA-seq count matrix (e.g. Smart-Seq2 primary visual cortex data from Tasic et al.)
- cell-type/state labels for each cell and isoform-to-gene mappings
- optionally, downsampled count matrices (e.g. 45 cells per neural type, 50 random draws) awaiting zero-inflation correction

## Outputs

- ZinBWaVE weight matrix with the same dimensions as the input count matrix, consumed by the acorde DIU workflow
- downstream per-round DIU results (DE isoform counts from DESeq2 and edgeR) aggregated into a summary such as downsampling_robustness_metrics.csv

## How to apply

Take the raw isoform-level count matrix (gene-to-isoform relationships and cell-type labels available, as in the acorde example data from Tasic et al. Smart-Seq2 primary visual cortex) and compute ZinBWaVE weights, which model zero inflation from both biological and technical sources and down-weight likely artificial zeros. Perform this weighting independently for each replicate of the experiment design — in the reproducibility benchmark, once per each of 50 random draws of 45 cells per neural type. Pass the weighted count matrices to the acorde DIU workflow and run multi-group DIU testing separately with DESeq2 and edgeR, rather than testing unweighted counts. Weighting should be seen as a variance/zero-model correction step preceding DE testing, not a filtering step: isoforms are retained in the matrix and their zero observations are penalized proportionally. Judge success by whether downstream DIU results meet the reported benchmarks (DESeq2 6908±101 DE isoforms, J=0.84±0.02; edgeR 6016±410, J=0.74±0.02) and by stability of counts/Jaccard indices across downsampling rounds.

## Related tools

- **ZinBWaVE** (computes zero-inflated weights applied to downsampled isoform count matrices before DIU testing)
- **acorde** (provides the DIU/coDIU workflow that consumes the weighted single-cell isoform-level expression matrix) — https://github.com/ConesaLab/acorde
- **DESeq2** (multi-group DIU testing on ZinBWaVE-weighted downsampled datasets (one of two methods benchmarked))
- **edgeR** (second multi-group DIU testing method run on the same weighted downsampled datasets)
- **R** (implementation language for the acorde DIU pipeline and weighting step) — https://github.com/ConesaLab/acorde

## Examples

```
devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE)
```

## Evaluation signals

- Weight matrix dimensions exactly match the input isoform x cell count matrix, with entries in [0,1].
- After weighted DIU testing across 50 downsampling rounds (45 cells/type), DESeq2 yields 6908±101 DE isoforms with mean Jaccard 0.84±0.02, and edgeR yields 6016±410 with J=0.74±0.02.
- Union-of-methods benchmark reached: 9399±248 DE isoforms with J=0.82±0.01.
- Summary metrics (counts and within-method Jaccard indices) are reproducible as averages over the 50 rounds and written to downsampling_robustness_metrics.csv.
- Results are computed on weighted matrices — running the same DIU tests on unweighted counts should degrade agreement with the reported benchmarks.

## Limitations

- Benchmark values were established for this specific dataset and design (Tasic neural cells, 45-cell downsampling, 50 rounds); other datasets or sample sizes will not match these counts exactly.
- acorde's bundled example data covers the manuscript's pipeline, but functional-annotation-dependent downstream steps are limited to pre-annotated references via isoAnnotLite.
- acorde requires isoform-level quantification with gene relationships and cell labels; expression matrices lacking these objects cannot enter the weighted DIU workflow.

## Evidence

- [readme] acorde DIU input requirements: "*acorde* contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input."
- [intro] DIU across multiple cell types: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell"
- [intro] source dataset: "scRNA-seq Smart-Seq2 (primary visual cortex, generated by Tasic et al.43,44)"
- [abstract] pipeline scope: "a pipeline that successfully leverages bulk long reads and single-cell data to confidently detect alternative isoform co-expression relationships"
- [readme] input flexibility: [evidence span withheld — anchor status: external_doc]
