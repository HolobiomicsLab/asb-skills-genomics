---
name: methylation-frequency-correlation-analysis
description: Use when you have Dorado call_mods-style per-site methylation_rate TSV
  output plus BS-seq methylation frequencies and need to tune a score threshold or
  benchmark site-level methylation quantification accuracy.
when_to_use_negative:
- When evaluating single-molecule accuracy, F1 score, or per-read recall — this skill
  measures population-level methylation frequency correlation only
- When no BS-seq methylation frequency reference exists for the species, since Pearson
  correlation with BS-seq is the sole accuracy metric
- When calls are already thresholded or filtered by a fixed confidence cutoff — re-binning
  0.2–0.9 to find a trough would rederive a threshold that no longer applies
tools:
- name: Dorado
  role: Produces the per-site methylation_rate calls (call_mods-style process) that
    are thresholded at P_th and correlated against BS-seq
  repo: nanoporetech/dorado
- name: DeepPlant
  role: Source of the adaptive methylation score threshold selection method (distribution-trough
    P_th) being applied to Dorado calls
  repo: github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:78994531874727b9c3548cd2dbcb254c5ec48af18a61b14dcc016b9417e53d98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - DeepPlant
  - Dorado
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# methylation-frequency-correlation-analysis

## Summary

Quantifies nanopore 5mC detection quality at population level by binning Dorado per-site methylation_rate values to pick an adaptive confidence threshold P_th, then computing whole-genome Pearson correlations between thresholded methylation frequencies and BS-seq across species and CHH contexts (the Dorado* condition of DeepPlant's Table 1).

## When to use

Use when you have Dorado call_mods-style per-site methylation_rate TSV output plus BS-seq methylation frequencies and need to tune a score threshold or benchmark site-level methylation quantification accuracy.

## When NOT to use

- When evaluating single-molecule accuracy, F1 score, or per-read recall — this skill measures population-level methylation frequency correlation only
- When no BS-seq methylation frequency reference exists for the species, since Pearson correlation with BS-seq is the sole accuracy metric
- When calls are already thresholded or filtered by a fixed confidence cutoff — re-binning 0.2–0.9 to find a trough would rederive a threshold that no longer applies

## Inputs

- Dorado call_mods-style output: TSV with per-site methylation_rate for called cytosines
- BS-seq methylation frequency table per genomic window and context (CpG/CHG/CHH) for the same species
- Reference genome and genomic window definitions for frequency aggregation

## Outputs

- Adaptive confidence threshold P_th (left endpoint of the minimum-population bin)
- Filtered per-site methylation calls (methylation_rate >= P_th)
- Per-window/context methylation frequency table for Dorado* calls
- Species/context table of whole-genome Pearson correlations with BS-seq (Dorado* rows of Table 1)

## How to apply

Load the Dorado methylation call TSV and bin all called cytosine sites by methylation_rate into 70 equal bins of step 0.01 spanning 0.2–0.9, counting sites per bin; the methylation score distribution is expected to be bimodal (methylated vs unmethylated), so set P_th to the left endpoint of the minimum-population bin, the trough that best separates the two modes. Retain only calls with methylation_rate >= P_th, then aggregate methylation frequency per genomic window and per cytosine context (CpG, CHG, CHH). Compute Pearson correlations between the thresholded Dorado* frequencies and BS-seq methylation frequencies for each species/context combination across the nine evaluated species. Judge success by reproducing the published Dorado* rows within rounding tolerance and confirming the improvement band over Dorado defaults (0.705–0.881 correlations, 14.0%–117.6% improvement for CHH). The same adaptive thresholding logic originates from DeepPlant and is applied to Dorado calls here to make the comparison against DeepPlant fair.

## Related tools

- **Dorado** (Produces the per-site methylation_rate calls (call_mods-style process) that are thresholded at P_th and correlated against BS-seq) — nanoporetech/dorado
- **DeepPlant** (Source of the adaptive methylation score threshold selection method (distribution-trough P_th) being applied to Dorado calls) — github.com/xiaochuanle/DeepPlant

## Examples

```
import numpy as np; hist, edges = np.histogram(meth_rates, bins=70, range=(0.2, 0.9)); p_th = edges[np.argmin(hist)]; corr = np.corrcoef(freq[p_th_mask], bsseq_freq)[0,1]
```

## Evaluation signals

- P_th equals the left endpoint of the minimum-population bin among 70 bins of step 0.01 spanning methylation_rate 0.2–0.9
- Every retained call satisfies methylation_rate >= P_th (invariant on the filtered TSV)
- Computed Dorado* Pearson correlations match the published Dorado* rows of Table 1 within rounding tolerance
- Correlation values fall within the reported band 0.705 to 0.881, with improvement over Dorado defaults within 14.0% to 117.6%
- Correlations are computed separately per species and per context (CpG/CHG/CHH), not pooled

## Limitations

- DeepPlant, whose threshold logic is reused here, is less computationally efficient than Dorado — it took much longer to call methylation on the same dataset (Supplementary Data 10)
- CHH remains the hardest context: scarcity of high-methylation CHH samples continues to pose challenges for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- Potential for overfitting to specific sequence patterns underscores the importance of careful model optimization and validation

## Evidence

- [abstract] Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado: "DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [intro] which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification: "In parallel, we develop DeepPlant, which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification."
- [methods] docker pull chenhx26/deepplant:1.1.0: "docker pull chenhx26/deepplant:1.1.0"
- [readme] Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing: "Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing."
