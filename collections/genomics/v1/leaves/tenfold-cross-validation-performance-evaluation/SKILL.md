---
name: tenfold-cross-validation-performance-evaluation
description: Use when per-read methylation scores (e.g. Megalodon, DeepSignal) or
  consensus predictions must be converted to per-site methylation frequencies on a
  mixture dataset of known proportion and scored against expected frequencies.
when_to_use_negative:
- The input has no ground-truth methylation frequencies (e.g. wild-type samples without
  controlled mixtures or WGBS reference), so expected-frequency-based metrics like
  RMSE and the 10% window proportion cannot be computed
- The analysis targets raw per-read methylation scores rather than aggregated per-site
  methylation frequency predictions
- The goal is comparing individual tools against each other without a trained consensus
  model, where a different benchmarking protocol applies
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: METEORE
  role: Provides the random forest and RidgeCV consensus models whose predictions
    are converted to per-site frequencies and evaluated with Pearson r, r^2, RMSE,
    and tenfold cross-validated ROC/PR curves
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Individual per-read methylation predictor whose per-site performance on the
    mixture dataset serves as the baseline the consensus must beat in RMSE
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Second individual per-read predictor combined by METEORE and used as an individual-tool
    baseline in the evaluation
  repo: https://github.com/bioinfomaticsCSU/deepsignal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:1c20ac8b4ca1ee40fb0736c171b12b136ac160afddf167a95bd39c57e0176c5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepSignal
  - METEORE
  - Megalodon
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# tenfold-cross-validation-performance-evaluation

## Summary

Evaluates per-site CpG methylation frequency predictions from nanopore methylation tools and METEORE consensus models on a controlled mixture dataset, using Pearson r, r^2, RMSE, ROC and precision-recall curves derived from tenfold cross-validation, plus the proportion of sites within the expected 10% window of the known mixture proportion.

## When to use

Use when per-read methylation scores (e.g. Megalodon, DeepSignal) or consensus predictions must be converted to per-site methylation frequencies on a mixture dataset of known proportion and scored against expected frequencies.

## When NOT to use

- The input has no ground-truth methylation frequencies (e.g. wild-type samples without controlled mixtures or WGBS reference), so expected-frequency-based metrics like RMSE and the 10% window proportion cannot be computed
- The analysis targets raw per-read methylation scores rather than aggregated per-site methylation frequency predictions
- The goal is comparing individual tools against each other without a trained consensus model, where a different benchmarking protocol applies

## Inputs

- Per-read score files (ID/Chr/Pos/Strand/Score) from Megalodon for mixture dataset 2
- Per-read score files (ID/Chr/Pos/Strand/Score) from DeepSignal for mixture dataset 2
- Trained METEORE random forest model (rf_model_max_depth_3_n_estimator_10) and RidgeCV regression model trained on mixture dataset 1
- Known mixture proportions (expected methylation frequencies) for mixture dataset 2

## Outputs

- Per-site methylation frequency table after thresholding Prob_methylation at 0.5
- Pearson r, r^2, and RMSE between predicted and expected per-site methylation frequencies
- ROC and precision-recall curves from tenfold cross-validation
- Proportion of sites predicted within the expected 10% window of the known mixture proportion

## How to apply

Train the METEORE random forest (rf_model_max_depth_3_n_estimator_10, n_estimators=10, max_depth=3) and RidgeCV regression models on mixture dataset 1 per-read scores, then apply both to mixture dataset 2 via combination_model_prediction.py (-m optimized). Convert per-read predictions to per-site methylation frequency with prediction_to_mod_frequency.py using a threshold of 0.5 on Prob_methylation. Compute Pearson r, r^2, RMSE between predicted and expected per-site methylation frequency, and generate ROC and precision-recall curves from tenfold cross-validation. Additionally report the proportion of sites predicted within the expected 10% window of the known mixture proportion. The consensus is judged successful only if the METEORE random forest achieves lower RMSE than either Megalodon or DeepSignal alone.

## Related tools

- **METEORE** (Provides the random forest and RidgeCV consensus models whose predictions are converted to per-site frequencies and evaluated with Pearson r, r^2, RMSE, and tenfold cross-validated ROC/PR curves) — https://github.com/comprna/METEORE
- **Megalodon** (Individual per-read methylation predictor whose per-site performance on the mixture dataset serves as the baseline the consensus must beat in RMSE) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Second individual per-read predictor combined by METEORE and used as an individual-tool baseline in the evaluation) — https://github.com/bioinfomaticsCSU/deepsignal

## Examples

```
python combination_model_prediction.py -m optimized -i megalodon_deepsignal_scores.tsv -o consensus_predictions.tsv; python prediction_to_mod_frequency.py -i consensus_predictions.tsv -t 0.5 -o per_site_frequency.tsv
```

## Evaluation signals

- The METEORE random forest consensus achieves lower RMSE against expected methylation frequency than either Megalodon or DeepSignal alone on mixture dataset 2
- Pearson r and r^2 are reported between predicted and expected per-site methylation frequencies with plausible ranges (0 <= r <= 1)
- ROC and precision-recall curves are produced from tenfold cross-validation over the mixture dataset sites
- The proportion of sites within the expected 10% window improves at intermediate and high methylation sets relative to individual tools

## Limitations

- Individual tools show a tradeoff between false positives and false negatives and high dispersion with respect to expected methylation frequency, so absolute per-site agreement is imperfect even for the consensus
- Combining multiple tools may be impractical for routine analyses due to the running times of methods without GPU support
- The M.SssI methylated control has only ~95% efficiency, which can depress agreement with expected methylation frequency

## Evidence

- [abstract] based on the combination of the predictions from two or more tools that shows improved accuracy over individual tools: "We found that tools have a tradeoff between false positives and false negatives and present a high dispersion with respect to the expected methylation frequency values"
- [discussion] consensus models evaluated per site: "the predictions per site generally showed a high dispersion and did not generally agree with the expected methylation frequency"
- [discussion] baseline individual tool performance: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
- [readme] consensus model definition: "new predictive models (random forest and multiple linear regression) that combine the outputs from the tools to produce a consensus prediction with higher accuracy than the individual tools"
