---
name: runtime-scaling-curve-visualization
description: Use when benchmarking how a single-cell annotation method's runtime scales
  with the number of query cells, e.g. verifying that scMAGIC's speed-up setting keeps
  runtime nearly flat above 5000 query cells on the DGH reference.
when_to_use_negative:
- The goal is to compare annotation accuracy (e.g. precision/recall across the 86
  benchmark tests) rather than computational runtime scaling
- The query dataset has fewer than ~5000 cells, where scMAGIC's speed-up setting has
  not yet engaged and scaling behavior cannot be assessed
- The reference and query are not the DGH (Tasic) / HArc-ME (Campbell) pair, so the
  reported scaling behavior is not directly comparable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Method under test whose runtime scaling curve is measured; run with atlas
    = 'MCA' and num_threads = 10
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: SingleR
  role: Competing statistical-metric-based RBC method included in the runtime comparison
  repo: https://github.com/dviraran/SingleR
- name: scmap
  role: Competing statistical-metric-based RBC method included in the runtime comparison
  repo: https://github.com/hemberg-lab/scmap-shiny
- name: SciBet
  role: Competing statistical-metric-based RBC method; also provides scMAGIC's HVGene
    selection backend (method_HVGene)
  repo: https://github.com/PaulingLiu/scibet
- name: CHETAH
  role: Competing tree-based RBC method included in the runtime comparison
- name: scClassify
  role: Competing tree-based RBC method included in the runtime comparison
- name: scPred
  role: Competing machine-learning-based RBC method included in the runtime comparison
- name: SingleCellNet
  role: Competing machine-learning-based RBC method included in the runtime comparison
- name: scID
  role: Competing machine-learning-based RBC method included in the runtime comparison
- name: ggplot2
  role: Used to plot runtime versus number of query cells for all 14 methods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:2c6ae72c52998c462d26380699de26e0397e89d6b63008c40c201f53ec5ee61e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - CHETAH
  - SciBet
  - SingleCellNet
  - SingleR
  - ggplot2
  - scClassify
  - scID
  - scMAGIC
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# runtime-scaling-curve-visualization

## Summary

Reproduce the computational scaling comparison of scMAGIC against 13 competing reference-based classification (RBC) methods by measuring wall-clock annotation runtime at increasing query-cell counts and plotting runtime-versus-query-size curves (Supplementary Figure S25A).

## When to use

Use when benchmarking how a single-cell annotation method's runtime scales with the number of query cells, e.g. verifying that scMAGIC's speed-up setting keeps runtime nearly flat above 5000 query cells on the DGH reference.

## When NOT to use

- The goal is to compare annotation accuracy (e.g. precision/recall across the 86 benchmark tests) rather than computational runtime scaling
- The query dataset has fewer than ~5000 cells, where scMAGIC's speed-up setting has not yet engaged and scaling behavior cannot be assessed
- The reference and query are not the DGH (Tasic) / HArc-ME (Campbell) pair, so the reported scaling behavior is not directly comparable

## Inputs

- Reference expression matrix and labels (Tasic.Rdata, list.Ref$mat_exp and list.Ref$label, GSE95315, 1727 SMART-seq mouse visual cortex cells)
- Full query expression matrix and labels (Campbell.Rdata, list.target$mat_exp and list.target$label, GSE93374 Drop-seq HArc-ME cells, up to 20,000 cells)
- Query subsets at increasing cell counts (subsampled exp_sc_mat)
- MCA atlas used by scMAGIC for marker gene identification

## Outputs

- Wall-clock annotation runtime measurements per method per query size (table indexed by method and query-cell number)
- Runtime-versus-query-size scaling curves for all 14 methods reproducing Supplementary Figure S25A

## How to apply

Download the DGH reference (Tasic.Rdata, GSE95315, 1727 SMART-seq mouse primary visual cortex cells) and the full HArc-ME query dataset (Campbell.Rdata, GSE93374 Drop-seq mouse hypothalamic arcuate-median eminence cells) from the scMAGIC_scripts repository. Install scMAGIC via devtools::install_github('Drizzle-Zhang/scMAGIC') plus the 13 competing RBC methods (SingleR, scmap, SciBet, CHETAH, scClassify, scPred, SingleCellNet, scID, CaSTLe, SVMrejection, etc.) at their optimal benchmark parameter settings. Subsample exp_sc_mat at increasing query-cell counts up to 20,000 and, for each subset, run scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10) and each competitor, recording wall-clock runtime per method. Assemble runtimes into a table indexed by method and query size, then plot runtime versus number of query cells with ggplot2 to reproduce the scaling curves of Supplementary Figure S25A.

## Related tools

- **scMAGIC** (Method under test whose runtime scaling curve is measured; run with atlas = 'MCA' and num_threads = 10) — https://github.com/TianLab-Bioinfo/scMAGIC
- **SingleR** (Competing statistical-metric-based RBC method included in the runtime comparison) — https://github.com/dviraran/SingleR
- **scmap** (Competing statistical-metric-based RBC method included in the runtime comparison) — https://github.com/hemberg-lab/scmap-shiny
- **SciBet** (Competing statistical-metric-based RBC method; also provides scMAGIC's HVGene selection backend (method_HVGene)) — https://github.com/PaulingLiu/scibet
- **CHETAH** (Competing tree-based RBC method included in the runtime comparison)
- **scClassify** (Competing tree-based RBC method included in the runtime comparison)
- **scPred** (Competing machine-learning-based RBC method included in the runtime comparison)
- **SingleCellNet** (Competing machine-learning-based RBC method included in the runtime comparison)
- **scID** (Competing machine-learning-based RBC method included in the runtime comparison)
- **ggplot2** (Used to plot runtime versus number of query cells for all 14 methods)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10)
```

## Evaluation signals

- scMAGIC's runtime curve must exhibit the reported speed-up behavior: runtime increases little once the number of query cells exceeds 5000
- scMAGIC finishes classifying 20,000 query cells within 6 minutes when run with num_threads = 10
- Runtime table covers all 14 methods (scMAGIC plus 13 competitors) at each subsampled query size
- The plotted curves reproduce the qualitative shapes of Supplementary Figure S25A

## Limitations

- No version numbers are stated for ggplot2, devtools, remotes, reticulate, or numpy, so exact reproducibility of the runtime environment is uncertain
- On Windows, scMAGIC requires method_HVGene = 'SciBet_R' and the zwj-tina/scibetR fallback, which may alter runtime relative to the Linux benchmark
- scMAGIC requires a Python environment with numpy available via reticulate before running; a missing numpy module breaks execution

## Evidence

- [results] scMAGIC finishes classifying 20 000 query cells within 6 min, and its runtime increases little once the number of query cells exceeds 5000 because of its speed-up setting.: [evidence span withheld — anchor status: not_found]
- [supplementary] Plot runtime versus number of query cells for all 14 methods to reproduce the scaling curves of Supplementary Figure S25A.: [evidence span withheld — anchor status: not_found]
- [methods] output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4): "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [readme] The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq. The target dataset is from a study profiling mouse hypothalamic arcuate-median eminence complex cells by Drop-seq: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq. The target dataset is from a study profiling mouse hypothalamic arcuate-median eminence complex cells by"
- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
