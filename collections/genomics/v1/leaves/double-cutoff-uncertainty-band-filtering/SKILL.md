---
name: double-cutoff-uncertainty-band-filtering
description: Use when per-read methylation scores from tools like Nanopolish, DeepSignal,
  or Megalodon are available for controlled mixtures and you need to reduce per-site
  RMSE by discarding reads with ambiguous methylation status.
when_to_use_negative:
- You only have a single sample without controlled mixtures to derive the ROC-based
  cutoffs and uncertainty band from
- Read coverage is so low that discarding 10% of reads around the crossing point would
  leave sites with unreliable coverage
- The goal is per-read (single-molecule) calls rather than aggregated per-site methylation
  frequencies
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: METEORE
  role: Aggregates filtered per-read scores into per-site methylation frequencies
    and combines Megalodon and DeepSignal predictions (RF/REG) for further improvement
  repo: https://github.com/comprna/METEORE
- name: Nanopolish
  role: Source of per-read methylation scores to which the double-cutoff filtering
    is applied
- name: DeepSignal
  role: Source of per-read methylation scores to which the double-cutoff filtering
    is applied
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Megalodon
  role: Source of per-read methylation scores to which the double-cutoff filtering
    is applied
  repo: https://github.com/nanoporetech/megalodon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:188495ee06d67729bbf2a6d4e3e6239a3c83f707609d025ee9c26af1e115b211
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepSignal
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Double-cutoff uncertainty-band filtering

## Summary

A read-discarding strategy for nanopore methylation tools that derives two per-read score cutoffs from the ROC curves of controlled methylation mixtures and removes reads falling in the uncertain band between them, improving per-site methylation frequency accuracy over default cutoffs.

## When to use

Use when per-read methylation scores from tools like Nanopolish, DeepSignal, or Megalodon are available for controlled mixtures and you need to reduce per-site RMSE by discarding reads with ambiguous methylation status.

## When NOT to use

- You only have a single sample without controlled mixtures to derive the ROC-based cutoffs and uncertainty band from
- Read coverage is so low that discarding 10% of reads around the crossing point would leave sites with unreliable coverage
- The goal is per-read (single-molecule) calls rather than aggregated per-site methylation frequencies

## Inputs

- Per-read methylation score files for Nanopolish, DeepSignal, and Megalodon from the figshare controlled mixture datasets (11 mixture proportions, ~2400 reads per set)
- Fully methylated and fully unmethylated control read score distributions for cutoff calibration
- Known per-site expected methylation frequencies for each mixture proportion

## Outputs

- Filtered per-read call sets with reads near the crossing point or inside the uncertainty band removed
- Per-site methylation frequency tables per tool/strategy combination at each mixture proportion
- RMSE and correlation metrics vs expected methylation frequency, tabulated as improvements over default cutoffs (cutoff_optimization_mixture2_metrics.csv)

## How to apply

On a fully methylated vs fully unmethylated mixture dataset (dataset 1 from the figshare deposit), build per-read score distributions and scan thresholds to locate, per tool, the crossing point of the FPR and 1-TPR curves plus an uncertainty band where FPR<0.05 and 1-TPR<0.05. Apply two cutoffs to the held-out mixture dataset 2 per-read scores: remove the 10% of reads whose scores are closest to the FPR/1-TPR crossing point, or remove reads falling inside the FPR<0.05 / 1-TPR<0.05 uncertainty band. Aggregate the retained per-read calls into per-site methylation frequencies at each of the 11 known mixture proportions (0% to 100% methylated, ~2400 reads per set) and compute RMSE and correlation against the expected methylation frequency relative to each tool's default cutoff. The rationale is that reads with intermediate scores carry uncertain methylation status, and discarding them (analogous to METEORE's --filter parameter that removes a percentage of reads around the mean score) lowers dispersion with respect to expected frequencies.

## Related tools

- **METEORE** (Aggregates filtered per-read scores into per-site methylation frequencies and combines Megalodon and DeepSignal predictions (RF/REG) for further improvement) — https://github.com/comprna/METEORE
- **Nanopolish** (Source of per-read methylation scores to which the double-cutoff filtering is applied)
- **DeepSignal** (Source of per-read methylation scores to which the double-cutoff filtering is applied) — https://github.com/bioinfomaticsCSU/deepsignal
- **Megalodon** (Source of per-read methylation scores to which the double-cutoff filtering is applied) — https://github.com/nanoporetech/megalodon

## Examples

```
python meteore combine --scores megalodon.tsv deepsignal.tsv --model RF --filter 10
```

## Evaluation signals

- At least one optimized double-cutoff strategy yields lower RMSE against expected methylation frequency than the tool's default cutoff on mixture dataset 2
- Pearson correlation with expected methylation frequency increases relative to the default-cutoff baseline
- Filtering removes exactly ~10% of reads nearest the FPR/1-TPR crossing point, or only reads with FPR<0.05 and 1-TPR<0.05
- Per-site frequencies at the 0% and 100% mixture points move toward the true extremes after filtering
- METEORE (RF/REG) combining Megalodon and DeepSignal after filtering outperforms each individual filtered tool

## Limitations

- Cutoffs derived from mixture dataset 1 may not transfer to datasets with different models, chemistries, or M.SssI methylation efficiency (the methylated control has only ~95% efficiency)
- Discarding reads reduces coverage, and combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support

## Evidence

- [discussion] Advantage of removing sites with uncertain methylation status: "Furthermore, there was an advantage in removing sites with uncertain methylation status."
- [methods] Read removal around mean score in METEORE: "--filter FLOAT       Percentage of total reads to remove around mean score, default -1 (disabled)"
- [methods] Training data from controlled mixture per-read scores on figshare: "Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264"
- [discussion] High dispersion motivates filtering: "the predictions per site generally showed a high dispersion and did not generally agree with the expected methylation frequency"
- [readme] METEORE combines tool outputs for improved accuracy: "new predictive models (random forest and multiple linear regression) that combine the outputs from the tools to produce a consensus prediction with higher accuracy than the individual tools."
