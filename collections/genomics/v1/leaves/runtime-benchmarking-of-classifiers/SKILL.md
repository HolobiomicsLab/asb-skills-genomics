---
name: runtime-benchmarking-of-classifiers
description: Use when quantifying how a single-cell annotation method's runtime scales
  with the number of query cells (e.g. subsampling from 100 up to 20,000 cells) and
  comparing that scaling against competing RBC classifiers.
when_to_use_negative:
- The goal is to compare annotation accuracy rather than runtime — use the paper's
  86 accuracy benchmark tests instead
- The query dataset cannot be paired with the DGH (Tasic) reference and HArc-ME (Campbell)
  query pair that anchors this scaling comparison
- Timing Windows-specific behavior is needed — scMAGIC requires method_HVGene = 'SciBet_R'
  on Windows, which changes the code path being timed
tools:
- name: scMAGIC
  role: Method under test; run per query subset with atlas = 'MCA' and num_threads
    = 10, timing wall-clock runtime
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: SingleR
  role: Competing statistical-metric-based RBC method included in the 14-method runtime
    comparison
  repo: https://github.com/dviraran/SingleR
- name: scmap
  role: Competing statistical-metric-based RBC method included in the runtime scaling
    comparison
  repo: https://github.com/hemberg-lab/scmap-shiny
- name: SciBet
  role: Competing statistical-metric-based RBC method; also backs scMAGIC's HVGene
    selection (method_HVGene)
  repo: https://github.com/PaulingLiu/scibet
- name: CHETAH
  role: Competing tree-based RBC method included in the runtime benchmark
  repo: https://github.com/jdekanter/CHETAH
- name: scClassify
  role: Competing tree-based RBC method included in the runtime benchmark
  repo: https://github.com/SydneyBioX/scClassify_analysis
- name: scPred
  role: Competing machine-learning-based RBC method included in the runtime benchmark
  repo: https://github.com/powellgenomicslab/scPred
- name: SingleCellNet
  role: Competing machine-learning-based RBC method included in the runtime benchmark
  repo: https://github.com/CahanLab/singleCellNet
- name: scID
  role: Competing machine-learning-based RBC method included in the runtime benchmark
  repo: https://github.com/BatadaLab/scID
- name: ggplot2
  role: Used to plot runtime versus number of query cells for the 14 methods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:1e84fcc7a649486881c1d0a66b42151b1764052f4d4a296813c5320121d2dada
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# runtime-benchmarking-of-classifiers

## Summary

Benchmark wall-clock annotation runtime of scMAGIC against 13 competing reference-based classification (RBC) methods across increasing query-cell counts (up to 20,000) using the DGH reference (Tasic, SMART-seq mouse visual cortex) and the HArc-ME query dataset (Campbell, Drop-seq), reproducing the scaling curves of Supplementary Figure S25A.

## When to use

Use when quantifying how a single-cell annotation method's runtime scales with the number of query cells (e.g. subsampling from 100 up to 20,000 cells) and comparing that scaling against competing RBC classifiers.

## When NOT to use

- The goal is to compare annotation accuracy rather than runtime — use the paper's 86 accuracy benchmark tests instead
- The query dataset cannot be paired with the DGH (Tasic) reference and HArc-ME (Campbell) query pair that anchors this scaling comparison
- Timing Windows-specific behavior is needed — scMAGIC requires method_HVGene = 'SciBet_R' on Windows, which changes the code path being timed

## Inputs

- Reference expression matrix and labels (Tasic.Rdata from GSE95315: list.Ref$mat_exp, list.Ref$label[, 1]; 1727 SMART-seq mouse visual cortex cells)
- Full query expression matrix (Campbell.Rdata from GSE93374, HArc-ME Drop-seq cells) from Drizzle-Zhang/scMAGIC_scripts
- Query subset sizes ranging up to 20,000 cells
- Installed R packages: scMAGIC plus 13 competing RBC methods at optimal benchmark parameters

## Outputs

- Table of wall-clock annotation runtimes indexed by method (14 methods) and query-cell count
- Runtime-versus-query-size scaling curves reproducing Supplementary Figure S25A (e.g. ggplot2)

## How to apply

Download the DGH reference dataset (GSE95315, Tasic.Rdata, 1727 mouse primary visual cortex cells profiled by SMART-seq) and the full HArc-ME query dataset (GSE93374, Campbell.Rdata) from the Drizzle-Zhang/scMAGIC_scripts repository. Install scMAGIC via devtools::install_github('Drizzle-Zhang/scMAGIC') together with the 13 competitors (SingleR, scmap, SciBet, CHETAH, scClassify, scPred, SingleCellNet, scID, CaSTLe, SVMrejection, etc.), each configured at its optimal parameter setting from the paper's 86 benchmark tests. Subsample the query expression matrix (exp_sc_mat) at increasing numbers of query cells up to 20,000, and for each subset run scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10) and each competitor, recording wall-clock annotation runtime per method and per size. Assemble the timings into a table indexed by method and query size, then plot runtime versus number of query cells for all 14 methods. The key expectation is that scMAGIC finishes classifying 20,000 query cells within 6 min and its runtime plateaus once the query count exceeds 5000 because of its speed-up setting.

## Related tools

- **scMAGIC** (Method under test; run per query subset with atlas = 'MCA' and num_threads = 10, timing wall-clock runtime) — https://github.com/TianLab-Bioinfo/scMAGIC
- **SingleR** (Competing statistical-metric-based RBC method included in the 14-method runtime comparison) — https://github.com/dviraran/SingleR
- **scmap** (Competing statistical-metric-based RBC method included in the runtime scaling comparison) — https://github.com/hemberg-lab/scmap-shiny
- **SciBet** (Competing statistical-metric-based RBC method; also backs scMAGIC's HVGene selection (method_HVGene)) — https://github.com/PaulingLiu/scibet
- **CHETAH** (Competing tree-based RBC method included in the runtime benchmark) — https://github.com/jdekanter/CHETAH
- **scClassify** (Competing tree-based RBC method included in the runtime benchmark) — https://github.com/SydneyBioX/scClassify_analysis
- **scPred** (Competing machine-learning-based RBC method included in the runtime benchmark) — https://github.com/powellgenomicslab/scPred
- **SingleCellNet** (Competing machine-learning-based RBC method included in the runtime benchmark) — https://github.com/CahanLab/singleCellNet
- **scID** (Competing machine-learning-based RBC method included in the runtime benchmark) — https://github.com/BatadaLab/scID
- **ggplot2** (Used to plot runtime versus number of query cells for the 14 methods)

## Examples

```
library(scMAGIC); output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10)
```

## Evaluation signals

- scMAGIC classifies 20,000 query cells within 6 minutes of wall-clock time
- scMAGIC's runtime curve increases little once the number of query cells exceeds 5000 (speed-up setting behavior)
- The runtime table has one row per method (14 methods total) with timings recorded at each query-cell subset size
- The plotted scaling curves reproduce Supplementary Figure S25A of the scMAGIC paper
- All 13 competitors were run at their optimal parameter settings established in the paper's benchmark tests, with a fixed num_threads = 10 for scMAGIC

## Limitations

- Competing methods must be configured at their optimal parameter settings from the paper's benchmark tests; deviating parameters invalidate the comparison
- Timings are hardware- and thread-dependent (scMAGIC run with num_threads = 10); absolute runtimes are not portable across machines
- Tool versions (Seurat, R, ggplot2, reticulate, numpy, etc.) are not pinned in the context, so exact reproduction may require version matching
- On Windows, method_HVGene = 'SciBet_R' must be set before timing scMAGIC, changing the measured code path

## Evidence

- [other] scMAGIC finishes classifying 20 000 query cells within 6 min, and its runtime increases little once the number of query cells exceeds 5000: [evidence span withheld — anchor status: not_found]
- [other] Run scMAGIC and each of the 13 competitors per query subset, recording wall-clock runtime: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation: scMAGIC runtime curve must exhibit speed-up above 5000 query cells (Supplementary Figure S25A): [evidence span withheld — anchor status: not_found]
- [methods] Reference dataset composition: 1727 SMART-seq mouse primary visual cortex cells: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq. The target dataset is from a study profiling mouse hypothalamic arcuate-median eminence complex cells by"
- [readme] scMAGIC install and invocation used for timing runs: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10)"
