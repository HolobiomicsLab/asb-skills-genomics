---
name: roc-based-cutoff-selection
description: Use when per-read methylation score files from nanopore tools must be
  aggregated into per-site methylation frequencies and the tools' default score cutoffs
  need replacing by cutoffs optimized on labeled control mixtures.
when_to_use_negative:
- The aim is benchmarking raw per-read detection accuracy (the TPR/FPR tradeoff itself)
  rather than producing per-site methylation frequencies with an optimized cutoff
- No labeled fully methylated / fully unmethylated control reads are available to
  estimate TPR and FPR distributions, so ROC-based cutoffs cannot be derived
- A caller other than Nanopolish, DeepSignal, or Megalodon is being analyzed without
  its own per-read score distributions from a mixture dataset
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: METEORE
  role: Consensus model (RF/REG) combining Nanopolish/DeepSignal/Megalodon per-read
    scores; supports a --filter option to remove a percentage of reads around the
    mean score
  repo: https://github.com/comprna/METEORE
- name: Nanopolish
  role: Source of per-read CpG methylation scores whose cutoff is optimized via ROC
    analysis
- name: DeepSignal
  role: Source of per-read CpG methylation scores whose cutoff is optimized via ROC
    analysis
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Megalodon
  role: Best-performing individual tool whose per-read score cutoff is optimized via
    ROC analysis
  repo: https://github.com/nanoporetech/megalodon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:1c7232545263f3c3b5ba45d3496fef71dae7172c910858c5417e4e1a01f77b7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepSignal
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# roc-based-cutoff-selection

## Summary

A calibration skill for nanopore 5mC detection tools (Nanopolish, DeepSignal, Megalodon, METEORE) that derives per-read score cutoffs from ROC analysis of fully methylated vs. fully unmethylated reads in a controlled mixture dataset, then applies them to independent mixtures to lower per-site RMSE versus default cutoffs.

## When to use

Use when per-read methylation score files from nanopore tools must be aggregated into per-site methylation frequencies and the tools' default score cutoffs need replacing by cutoffs optimized on labeled control mixtures.

## When NOT to use

- The aim is benchmarking raw per-read detection accuracy (the TPR/FPR tradeoff itself) rather than producing per-site methylation frequencies with an optimized cutoff
- No labeled fully methylated / fully unmethylated control reads are available to estimate TPR and FPR distributions, so ROC-based cutoffs cannot be derived
- A caller other than Nanopolish, DeepSignal, or Megalodon is being analyzed without its own per-read score distributions from a mixture dataset

## Inputs

- Per-read score files from Nanopolish, DeepSignal, and Megalodon for controlled mixture dataset 1 (figshare deposit 15081264) and mixture dataset 2 (11 mixture proportions, ~2400 reads per set)
- Known per-set methylation proportions (0%-100%) used as expected methylation frequencies

## Outputs

- Per-tool optimized single cutoffs (max TPR-FPR and min FPR^2+(1-TPR)^2) and two double-cutoff read-discarding rules
- Per-site methylation frequency calls per tool/strategy/mixture proportion
- RMSE and correlation metrics table: cutoff_optimization_mixture2_metrics.csv

## How to apply

Download per-read score files (figshare deposit of controlled mixture datasets, 11 mixture proportions from 0% to 100% methylated, ~2400 reads per set) for each tool. On mixture dataset 1, build per-read score distributions for fully methylated and fully unmethylated reads and scan thresholds to select, per tool, the single cutoff maximizing TPR-FPR and the cutoff minimizing FPR^2+(1-TPR)^2. Derive two double-cutoff strategies per tool: discard the 10% of reads with scores closest to the FPR/(1-TPR) crossing point, and discard reads in the FPR<0.05 / 1-TPR<0.05 uncertainty band. Apply each optimized single- and double-cutoff strategy, plus the default-cutoff baseline, to the mixture dataset 2 per-read scores and aggregate reads into per-site methylation frequencies at each known mixture proportion. Compute RMSE and correlation against expected methylation frequency per tool/strategy combination, writing the combined table to cutoff_optimization_mixture2_metrics.csv; the rationale is that removing reads of uncertain methylation status improves per-site accuracy.

## Related tools

- **METEORE** (Consensus model (RF/REG) combining Nanopolish/DeepSignal/Megalodon per-read scores; supports a --filter option to remove a percentage of reads around the mean score) — https://github.com/comprna/METEORE
- **Nanopolish** (Source of per-read CpG methylation scores whose cutoff is optimized via ROC analysis)
- **DeepSignal** (Source of per-read CpG methylation scores whose cutoff is optimized via ROC analysis) — https://github.com/bioinfomaticsCSU/deepsignal
- **Megalodon** (Best-performing individual tool whose per-read score cutoff is optimized via ROC analysis) — https://github.com/nanoporetech/megalodon

## Evaluation signals

- For the best-performing tools, at least one optimized (double-)cutoff strategy reduces RMSE and increases correlation with expected methylation frequency relative to the default cutoff on mixture dataset 2
- All tools show lower RMSE with the max-TPR-FPR single cutoff (derived on mixture dataset 1) than with their default cutoffs
- METEORE (RF/REG) combining Megalodon and DeepSignal further improves RMSE/correlation over the individual tools
- cutoff_optimization_mixture2_metrics.csv contains one metrics row per tool/strategy combination across the 11 mixture proportions
- Selected cutoffs reproduce the ROC-derived crossing points (FPR<0.05 / 1-TPR<0.05 band) from mixture dataset 1 score distributions

## Limitations

- The M.SssI treatment used to generate the CpG-methylated control only has ~95% efficiency, a possible reason for low agreement with expected methylation frequency
- Combining multiple tools (e.g. Megalodon + DeepSignal via METEORE) improved accuracy but may be impractical for routine analyses due to running times of methods without GPU support
- The METEORE repository for Megalodon is deprecated; production modified base calling has moved to Dorado/Remora

## Evidence

- [supplementary] Optimized single cutoff beats defaults: [evidence span withheld — anchor status: not_found]
- [discussion] Advantage of removing uncertain reads/sites: "Furthermore, there was an advantage in removing sites with uncertain methylation status."
- [readme] METEORE read-filtering parameter: "--filter FLOAT       Percentage of total reads to remove around mean score, default -1 (disabled)"
- [methods] Per-read score training data source: "you can download them from figshare (https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264)"
