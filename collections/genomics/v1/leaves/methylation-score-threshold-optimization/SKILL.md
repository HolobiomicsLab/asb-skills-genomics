---
name: methylation-score-threshold-optimization
description: Use when per-read methylation scores exist from a controlled mixture
  of methylated/unmethylated nanopore reads and default tool cutoffs show high dispersion
  against expected per-site methylation frequencies.
when_to_use_negative:
- Only per-site aggregated methylation frequencies are available and no per-read score
  files exist, since ROC-based cutoff derivation requires individual read scores from
  fully methylated and fully unmethylated controls
- The data are not controlled mixtures with known methylation proportions, so expected
  methylation frequencies for RMSE/correlation validation cannot be established
- The target modification or platform is not 5mC at CpG sites from R9.4.1 nanopore
  reads scored by these tools
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Nanopolish
  role: Source of per-read methylation scores whose default cutoff is benchmarked
    and whose score distributions are used to derive optimized thresholds
- name: DeepSignal
  role: Source of per-read scores for cutoff optimization; also combined with Megalodon
    via METEORE for consensus improvement
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Megalodon
  role: Best-performing individual tool whose per-read scores are re-thresholded;
    combined with DeepSignal by METEORE
  repo: https://github.com/nanoporetech/megalodon
- name: METEORE
  role: Random forest / multiple linear regression consensus of combined per-read
    scores that further improves per-site accuracy beyond single-tool optimized cutoffs
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:eecdf1aef8faa55d15807aa1c9447e24e5b5b2417cbe6212dc78231742803049
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinfomaticsCSU/deepsignal
  tools:
  - DeepSignal
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# methylation-score-threshold-optimization

## Summary

Optimizes per-read score cutoffs for nanopore CpG methylation tools (Nanopolish, DeepSignal, Megalodon) by scanning score distributions from controlled mixtures of fully methylated and fully unmethylated reads, deriving single and double cutoffs that outperform default thresholds and reduce RMSE of per-site methylation frequency predictions.

## When to use

Use when per-read methylation scores exist from a controlled mixture of methylated/unmethylated nanopore reads and default tool cutoffs show high dispersion against expected per-site methylation frequencies.

## When NOT to use

- Only per-site aggregated methylation frequencies are available and no per-read score files exist, since ROC-based cutoff derivation requires individual read scores from fully methylated and fully unmethylated controls
- The data are not controlled mixtures with known methylation proportions, so expected methylation frequencies for RMSE/correlation validation cannot be established
- The target modification or platform is not 5mC at CpG sites from R9.4.1 nanopore reads scored by these tools

## Inputs

- Per-read score files from Nanopolish, DeepSignal, and Megalodon for controlled mixture dataset 1 (figshare deposit, 11 mixture proportions, ~2400 reads per set)
- Per-read score files for the same tools on mixture dataset 2
- Expected per-site methylation frequencies at each known mixture proportion
- Reference genome annotations of CpG sites for aggregating per-read calls per site

## Outputs

- Per-tool optimized single cutoffs (max TPR-FPR and min FPR^2+(1-TPR)^2) and double-cutoff definitions
- Per-site methylation frequency predictions for every tool/strategy combination
- RMSE and correlation metrics vs expected methylation frequency per tool/strategy combination
- cutoff_optimization_mixture2_metrics.csv (combined cutoff and metrics table)

## How to apply

Download per-read score files for Nanopolish, DeepSignal, and Megalodon from the figshare deposit of the 11 controlled mixture proportions (0% to 100% methylated, ~2400 reads per set). On mixture dataset 1, build per-read score distributions for fully methylated and fully unmethylated controls and scan thresholds to select, per tool, the single cutoff maximizing TPR-FPR and the one minimizing FPR^2+(1-TPR)^2. Derive two double-cutoff strategies: remove the 10% of reads with scores closest to the FPR/1-TPR crossing point, and remove reads in the FPR<0.05 / 1-TPR<0.05 uncertainty band. Apply each optimized (double-)cutoff plus the default cutoff baseline to mixture dataset 2 scores, aggregate into per-site methylation frequencies, and compute RMSE and correlation against expected methylation frequency; optionally combine Megalodon and DeepSignal with METEORE random forest/regression models for further gains.

## Related tools

- **Nanopolish** (Source of per-read methylation scores whose default cutoff is benchmarked and whose score distributions are used to derive optimized thresholds)
- **DeepSignal** (Source of per-read scores for cutoff optimization; also combined with Megalodon via METEORE for consensus improvement) — https://github.com/bioinfomaticsCSU/deepsignal
- **Megalodon** (Best-performing individual tool whose per-read scores are re-thresholded; combined with DeepSignal by METEORE) — https://github.com/nanoporetech/megalodon
- **METEORE** (Random forest / multiple linear regression consensus of combined per-read scores that further improves per-site accuracy beyond single-tool optimized cutoffs) — https://github.com/comprna/METEORE

## Evaluation signals

- Each optimized single cutoff corresponds to the maximum TPR-FPR (or minimum FPR^2+(1-TPR)^2) point of the ROC curve computed on mixture dataset 1
- On mixture dataset 2, applying the optimized cutoff yields a lower RMSE and higher correlation with expected methylation frequency than the tool's default cutoff
- At least one double-cutoff strategy (10% removed near crossing point or FPR<0.05/1-TPR<0.05 band) reduces RMSE for the best-performing tools, reproducing the advantage of removing uncertain-status reads
- METEORE (RF/REG) combining Megalodon and DeepSignal scores achieves lower RMSE than either individual tool with optimized cutoffs
- All per-tool/strategy metrics are tabulated in cutoff_optimization_mixture2_metrics.csv with rows covering every combination tested

## Limitations

- Cutoffs derived on mixture dataset 1 may be dataset-specific; transferability was validated only on a second mixture dataset
- The M.SssI treatment used to generate the CpG-methylated control only has 95% efficiency, so expected methylation frequencies at 100% are imperfect references
- The METEORE consensus and Megalodon/DeepSignal combination require GPUs to be time efficient, which may be impractical for routine analyses

## Evidence

- [discussion] Advantage of removing uncertain sites: "Furthermore, there was an advantage in removing sites with uncertain methylation status."
- [abstract] METEORE consensus rationale: "based on the combination of the predictions from two or more tools that shows improved accuracy over individual tools"
- [discussion] Megalodon+DeepSignal combination: "The combination of Megalodon and DeepSignal struck a good balance for accuracy and running times but would require a GPU for efficiency."
- [readme] Per-read filtering option in METEORE: "--filter FLOAT       Percentage of total reads to remove around mean score, default -1 (disabled)"
- [readme] Score truncation parameters: "The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis"
