---
name: single-cell-rna-seq-query-subsampling
description: Use when benchmarking how a single-cell annotation method's runtime scales
  with query dataset size, or when reproducing runtime scaling curves (e.g., Supplementary
  Figure S25A of scMAGIC) across competing RBC methods.
when_to_use_negative:
- When the goal is annotation accuracy rather than runtime scaling — accuracy requires
  full-dataset or ground-truth-label benchmarks, not subsampled timing runs
- When the query dataset contains fewer than ~5,000 cells, since the speed-up behavior
  above 5,000 query cells will not be observable
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Method under test whose runtime scaling curve is measured; run with num_threads
    = 10 and atlas = 'MCA'
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: SingleR
  role: Competing statistical-metric-based RBC method timed at increasing query sizes
  repo: https://github.com/dviraran/SingleR
- name: scmap
  role: Competing statistical-metric-based RBC method timed at increasing query sizes
  repo: https://github.com/hemberg-lab/scmap-shiny
- name: SciBet
  role: Competing statistical-metric-based RBC method timed at increasing query sizes
  repo: https://github.com/PaulingLiu/scibet
- name: CHETAH
  role: Competing tree-based RBC method timed at increasing query sizes
  repo: https://github.com/jdekanter/CHETAH
- name: scClassify
  role: Competing tree-based RBC method timed at increasing query sizes
  repo: https://github.com/SydneyBioX/scClassify
- name: scPred
  role: Competing machine-learning-based RBC method timed at increasing query sizes
  repo: https://github.com/powellgenomicslab/scPred
- name: SingleCellNet
  role: Competing machine-learning-based RBC method timed at increasing query sizes
  repo: https://github.com/CahanLab/singleCellNet
- name: scID
  role: Competing machine-learning-based RBC method timed at increasing query sizes
  repo: https://github.com/BatadaLab/scID
- name: ggplot2
  role: Used to plot runtime versus number of query cells for all 14 methods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:1fca4f94959417cc3a361f3b442eab29ded9c9cbc3eb522d904c23d25deb65ef
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# single-cell-rna-seq-query-subsampling

## Summary

Subsamples an scRNA-seq query expression matrix at increasing cell counts (up to 20,000) and measures wall-clock annotation runtime per method, enabling runtime scaling comparison of reference-based classification methods such as scMAGIC against 13 competing RBC tools.

## When to use

Use when benchmarking how a single-cell annotation method's runtime scales with query dataset size, or when reproducing runtime scaling curves (e.g., Supplementary Figure S25A of scMAGIC) across competing RBC methods.

## When NOT to use

- When the goal is annotation accuracy rather than runtime scaling — accuracy requires full-dataset or ground-truth-label benchmarks, not subsampled timing runs
- When the query dataset contains fewer than ~5,000 cells, since the speed-up behavior above 5,000 query cells will not be observable

## Inputs

- Reference expression matrix and labels (Tasic.Rdata from GSE95315: 1727 mouse primary visual cortex SMART-seq cells)
- Full query expression matrix and labels (Campbell.Rdata from GSE93374: Drop-seq mouse hypothalamic arcuate-median eminence complex cells)
- R environment with scMAGIC and 13 competing RBC methods installed (SingleR, scmap, SciBet, CHETAH, scClassify, scPred, SingleCellNet, scID, CaSTLe, SVMrejection, etc.)

## Outputs

- Table of wall-clock annotation runtime indexed by method and query subset size
- Runtime versus number-of-query-cells scaling curves for all 14 methods (reproducing Supplementary Figure S25A)

## How to apply

Download the DGH reference dataset (GSE95315, Tasic.Rdata) and the full HArc-ME query dataset (GSE93374, Campbell.Rdata) from the scMAGIC_scripts repository. Subsample the query expression matrix (exp_sc_mat) at increasing numbers of query cells up to 20,000, then for each subset run scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10) alongside each of the 13 competitors configured at their optimal benchmark parameters, recording wall-clock runtime per method per size. Assemble runtimes into a table indexed by method and query size, and plot runtime versus number of query cells for all 14 methods. scMAGIC classifies 20,000 query cells within 6 minutes, and its runtime increases little once query size exceeds 5,000 cells due to its speed-up setting, which serves as the validation signature of the scaling curve.

## Related tools

- **scMAGIC** (Method under test whose runtime scaling curve is measured; run with num_threads = 10 and atlas = 'MCA') — https://github.com/TianLab-Bioinfo/scMAGIC
- **SingleR** (Competing statistical-metric-based RBC method timed at increasing query sizes) — https://github.com/dviraran/SingleR
- **scmap** (Competing statistical-metric-based RBC method timed at increasing query sizes) — https://github.com/hemberg-lab/scmap-shiny
- **SciBet** (Competing statistical-metric-based RBC method timed at increasing query sizes) — https://github.com/PaulingLiu/scibet
- **CHETAH** (Competing tree-based RBC method timed at increasing query sizes) — https://github.com/jdekanter/CHETAH
- **scClassify** (Competing tree-based RBC method timed at increasing query sizes) — https://github.com/SydneyBioX/scClassify
- **scPred** (Competing machine-learning-based RBC method timed at increasing query sizes) — https://github.com/powellgenomicslab/scPred
- **SingleCellNet** (Competing machine-learning-based RBC method timed at increasing query sizes) — https://github.com/CahanLab/singleCellNet
- **scID** (Competing machine-learning-based RBC method timed at increasing query sizes) — https://github.com/BatadaLab/scID
- **ggplot2** (Used to plot runtime versus number of query cells for all 14 methods)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10)
```

## Evaluation signals

- scMAGIC finishes classifying 20,000 query cells within 6 minutes
- scMAGIC's runtime curve exhibits a speed-up: runtime increases little once the number of query cells exceeds 5,000, matching Supplementary Figure S25A
- Runtime table has one row per method per query subset size, covering all 14 methods
- Each competitor is run at its optimal parameter setting established in the paper's benchmark tests

## Limitations

- Runtime scaling depends on num_threads (10 in the reference run) and on the speed-up setting built into scMAGIC, so results are not directly comparable across hardware or thread counts
- Subsampling changes the query cell composition, so timing runs are not suited to accuracy assessment
- No version numbers are stated for ggplot2, devtools, remotes, reticulate, or numpy, and no archived code release or Zenodo DOI exists for scMAGIC beyond the GitHub repository link
- Windows users must set method_HVGene = 'SciBet_R' when running scMAGIC, which may alter timing characteristics

## Evidence

- [other] scMAGIC finishes classifying 20 000 query cells within 6 min, and its runtime increases little once the number of query cells exceeds 5000 because of its speed-up setting.: [evidence span withheld — anchor status: not_found]
- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [readme] output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4): "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [methods] The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq.: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq."
