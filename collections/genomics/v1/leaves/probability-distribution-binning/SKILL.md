---
name: probability-distribution-binning
description: Use when you have per-site nanopore methylation calls with a continuous
  score (e.g., methylation_rate from Dorado call_mods output) and need a data-driven
  confidence cutoff before aggregating methylation frequency for benchmarking against
  BS-seq.
when_to_use_negative:
- Methylation calls already carry a validated fixed confidence threshold from the
  tool vendor and re-tuning is out of scope
- The score distribution over 0.2-0.9 is unimodal with no discernible trough, making
  the minimum-population bin an unstable threshold
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Dorado
  role: Produces the per-site methylation_rate calls (call_mods-style TSV) that the
    binning threshold is applied to
  repo: https://github.com/nanoporetech/dorado
- name: DeepPlant
  role: Source of the adaptive threshold-selection method (P_th from probability-distribution
    trough) that is transferred to Dorado calls
  repo: https://github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:a36581ac44a8b3af59d8f7e2b34cb8315e9231fde56be4a29061ab95bd5edb44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - DeepPlant
  - Dorado
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# probability-distribution-binning

## Summary

Selects an adaptive methylation score confidence threshold (P_th) by binning per-site methylation_rate values into a probability histogram and picking the trough (minimum-population bin) as the decision boundary. Borrowed from DeepPlant, it minimizes the sum of false-positive and false-negative methylation calls and was shown to improve Dorado's site-level methylation correlations with BS-seq (the Dorado* condition).

## When to use

Use when you have per-site nanopore methylation calls with a continuous score (e.g., methylation_rate from Dorado call_mods output) and need a data-driven confidence cutoff before aggregating methylation frequency for benchmarking against BS-seq.

## When NOT to use

- Methylation calls already carry a validated fixed confidence threshold from the tool vendor and re-tuning is out of scope
- The score distribution over 0.2-0.9 is unimodal with no discernible trough, making the minimum-population bin an unstable threshold

## Inputs

- Dorado call_mods-style TSV with per-site methylation_rate values
- Reference genome for assigning sites to genomic windows
- BS-seq methylation frequency table per window/context for correlation

## Outputs

- Adaptive confidence threshold P_th (float, left endpoint of the minimum-population bin)
- Filtered methylation call set (sites with methylation_rate >= P_th)
- Windowed methylation frequency table per context (e.g., CHH)
- Pearson correlation values vs BS-seq per species/context (Dorado* rows)

## How to apply

Load the Dorado methylation call TSV containing per-site methylation_rate values, then histogram all called cytosine sites across the 0.2-0.9 score range in 70 equal bins of step 0.01. Identify the bin with the minimum number of sites — this trough separates the low-score (unmethylated) and high-score (methylated) populations — and set P_th to that bin's left endpoint. Retain only calls with methylation_rate >= P_th, then aggregate methylation frequency per genomic window and sequence context (e.g., CHH). Compute whole-genome Pearson correlations of the thresholded frequencies against BS-seq methylation frequencies per species/context. The rationale is that the empirical score distribution is bimodal and the trough maximizes agreement, mimicking DeepPlant's adaptive threshold selection applied to Dorado calls (Table 1, Dorado* rows).

## Related tools

- **Dorado** (Produces the per-site methylation_rate calls (call_mods-style TSV) that the binning threshold is applied to) — https://github.com/nanoporetech/dorado
- **DeepPlant** (Source of the adaptive threshold-selection method (P_th from probability-distribution trough) that is transferred to Dorado calls) — https://github.com/xiaochuanle/DeepPlant

## Examples

```
import numpy as np; counts, edges = np.histogram(df['methylation_rate'], bins=np.arange(0.2, 0.91, 0.01)); P_th = edges[counts.argmin()]; kept = df[df['methylation_rate'] >= P_th]
```

## Evaluation signals

- Computed Dorado* Pearson correlation values reproduce the published Dorado* rows of Table 1 within rounding tolerance
- Derived improvement over default Dorado falls within the reported band: correlations 0.705-0.881 and 14.0%-117.6% improvement
- P_th equals the left endpoint of the minimum-population bin over 70 bins spanning 0.2-0.9
- Filtered call set contains only sites with methylation_rate >= P_th, and windowed frequency aggregation uses the same genomic windows and CHH context as the BS-seq comparison

## Limitations

- DeepPlant, from which this thresholding approach derives, is less computationally efficient than Dorado (longer runtime on the same dataset, Supplementary Data 10)
- Correlation improvement ranges (e.g., 23.4-117.6% on CHH) are species- and context-dependent; gains may not transfer to organisms with sparse CHH methylation
- Scarcity of high-methylation CHH samples in some plant species limits generalization of the score distribution shape the trough-based method assumes

## Evidence

- [abstract] DeepPlant CHH correlations abstract: "Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [intro] DeepPlant outperforms Dorado with correlation improvement: "In parallel, we develop DeepPlant, which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification."
- [full_text] Adaptive threshold applied to Dorado calls: [evidence span withheld — anchor status: not_found]
- [abstract] Dorado as the R10.4 plant 5mC tool: "Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing"
- [abstract] DeepPlant model description: "Here, we develop DeepPlant, a deep learning model incorporating both Bi-LSTM and Transformer architectures"
