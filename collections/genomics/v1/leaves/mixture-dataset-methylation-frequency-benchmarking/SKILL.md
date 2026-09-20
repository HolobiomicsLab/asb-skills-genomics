---
name: mixture-dataset-methylation-frequency-benchmarking
description: Use when per-read methylation scores from controlled mixtures of fully
  methylated and unmethylated reads are available and the goal is to test whether
  adjusted score cutoffs improve per-site methylation frequency prediction versus
  tool defaults.
when_to_use_negative:
- Input is raw fast5 signal data rather than pre-computed per-read methylation score
  files — methylation calling must be run first (e.g., via the METEORE Snakemake pipelines).
- There is no known ground-truth methylation frequency (e.g., whole-genome samples
  rather than controlled mixtures), so RMSE/correlation against expected values cannot
  be computed.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: METEORE
  role: Consensus (random forest / multiple linear regression) combination of per-read
    scores from Megalodon and DeepSignal to improve per-site methylation frequency
    accuracy beyond individual tools
  repo: https://github.com/comprna/METEORE
- name: Nanopolish
  role: Per-read CpG methylation score provider whose default vs optimized cutoffs
    are benchmarked on the mixture datasets
- name: DeepSignal
  role: Per-read CpG methylation score provider benchmarked with optimized cutoffs;
    also combined with Megalodon in METEORE consensus
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Megalodon
  role: Per-read CpG methylation score provider benchmarked with optimized cutoffs;
    best individual performer
  repo: https://github.com/nanoporetech/megalodon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:17178cb64bf5063b4b1b19b69bace5177defb38c4bd64d2a61afb65a507641d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepSignal
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# mixture-dataset-methylation-frequency-benchmarking

## Summary

Benchmark nanopore 5mC methylation-calling tools by applying optimized single and double per-read score cutoffs to controlled methylation mixtures and scoring per-site methylation frequencies against known expected frequencies via RMSE and correlation.

## When to use

Use when per-read methylation scores from controlled mixtures of fully methylated and unmethylated reads are available and the goal is to test whether adjusted score cutoffs improve per-site methylation frequency prediction versus tool defaults.

## When NOT to use

- Input is raw fast5 signal data rather than pre-computed per-read methylation score files — methylation calling must be run first (e.g., via the METEORE Snakemake pipelines).
- There is no known ground-truth methylation frequency (e.g., whole-genome samples rather than controlled mixtures), so RMSE/correlation against expected values cannot be computed.

## Inputs

- Per-read methylation score files for Nanopolish, DeepSignal, and Megalodon from controlled mixture datasets (11 mixture proportions 0-100%, ~2400 reads per set, figshare deposit)
- Expected methylation frequency per mixture proportion (known ground truth)
- METEORE combined per-read score file (Megalodon + DeepSignal) for RF/REG consensus predictions

## Outputs

- Per-site methylation frequencies for each tool/strategy combination at each mixture proportion
- Table of RMSE and correlation versus expected methylation frequency per tool and cutoff strategy
- cutoff_optimization_mixture2_metrics.csv (combined cutoff and metrics table)

## How to apply

Download per-read score files for Nanopolish, DeepSignal, and Megalodon from the figshare deposit of controlled mixture datasets (11 mixture proportions from 0% to 100% methylated reads, ~2400 reads per set). On mixture dataset 1, build per-read score distributions for fully methylated and fully unmethylated reads and scan thresholds to select, per tool, the single cutoff maximizing TPR-FPR (and the cutoff minimizing FPR^2+(1-TPR)^2). Derive double-cutoff strategies that discard uncertain reads: remove the 10% of reads with scores closest to the FPR/1-TPR crossing point, or remove reads in the FPR<0.05 / 1-TPR<0.05 uncertainty band. Apply each strategy plus the default-cutoff baseline to mixture dataset 2 per-read scores, aggregate reads into per-site methylation frequencies, and compute RMSE and correlation against expected frequencies; optionally include METEORE (random forest / linear regression) consensus predictions combining Megalodon and DeepSignal.

## Related tools

- **METEORE** (Consensus (random forest / multiple linear regression) combination of per-read scores from Megalodon and DeepSignal to improve per-site methylation frequency accuracy beyond individual tools) — https://github.com/comprna/METEORE
- **Nanopolish** (Per-read CpG methylation score provider whose default vs optimized cutoffs are benchmarked on the mixture datasets)
- **DeepSignal** (Per-read CpG methylation score provider benchmarked with optimized cutoffs; also combined with Megalodon in METEORE consensus) — https://github.com/bioinfomaticsCSU/deepsignal
- **Megalodon** (Per-read CpG methylation score provider benchmarked with optimized cutoffs; best individual performer) — https://github.com/nanoporetech/megalodon

## Examples

```
python combine_call_modification.py --scores meg_scores.txt,ds_scores.txt --model rf --output meteore_combined.bed
```

## Evaluation signals

- At least one optimized (double-)cutoff strategy reduces RMSE and increases correlation with expected methylation frequency relative to the tool's default cutoff for the best-performing tools.
- Cutoffs derived on mixture dataset 1 transfer to mixture dataset 2 (single cutoffs from the maximum TPR-FPR give lower RMSE than defaults).
- Aggregated per-site methylation frequencies fall near the expected mixture proportions (0-100%) with low dispersion.
- METEORE (RF/REG) consensus of Megalodon + DeepSignal yields lower RMSE than the individual tools.
- cutoff_optimization_mixture2_metrics.csv contains one metrics row per tool/strategy combination with finite RMSE and correlation values.

## Limitations

- The M.SssI treatment used to generate the CpG-methylated control has only ~95% efficiency, a possible reason for low agreement with expected methylation frequency.
- Combining multiple tools improves accuracy but may be impractical for routine analyses due to running times of methods without GPU support (e.g., Megalodon and DeepSignal rely on GPUs to be time efficient).
- Cutoffs are derived from one mixture dataset and validated on another; transferability to other organisms or sequencing chemistries is not established.

## Evidence

- [supplementary] applying single score cutoffs corresponding to the maximum TPR-FPR: [evidence span withheld — anchor status: not_found]
- [supplementary] remove the 10% of reads with scores closest to the FPR/1-TPR crossing point: [evidence span withheld — anchor status: not_found]
- [methods] controlled mixture datasets: "you can download them from figshare (https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264)"
- [discussion] advantage of removing uncertain sites: "Furthermore, there was an advantage in removing sites with uncertain methylation status."
- [discussion] Megalodon best performance: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
