---
name: greedy-forward-feature-selection
description: Use when a small-sample regression task (e.g., predicting ΔΨ on vex-seq
  TR957) has many candidate features (e.g., 42 splicing-related features) and you
  must identify the minimal subset maximizing cross-validated PCC.
when_to_use_negative:
- The input is already a small, curated feature set where subset selection would overfit
  the CV metric
- The sample size is large enough for embedded feature-selection methods (e.g., regularized
  or deep models); the paper notes the small number of vex-seq samples is what motivated
  greedy selection over more powerful algorithms
- The target task is classification rather than ΔΨ regression evaluated by PCC, and
  no analogous validation metric is available per subset
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3320
tools:
- name: PredPSI-SVR
  role: Framework whose 42-feature matrix and ε-SVR (LIBSVM RBF) regression are the
    objects of greedy forward selection; selects the 8-feature final model
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Provides the ε-SVR implementation used to score each candidate feature subset
    via 10-fold CV PCC
  repo: https://www.csie.ntu.edu.tw/~cjlin/libsvm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:27ca7aea52b5ba334a43c0b9f404cbe81d073198da7f7f8070c32c4969bee6f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - PredPSI-SVR
  - Python 3
  - libsvm
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# greedy-forward-feature-selection

## Summary

Greedy forward feature selection for high-dimensional ML: at each round add the single remaining feature that maximizes k-fold cross-validation performance, stopping when the metric (here Pearson correlation coefficient) peaks. Used in PredPSI-SVR to shrink 42 splicing-related features down to the eight that contribute most to ΔΨ prediction.

## When to use

Use when a small-sample regression task (e.g., predicting ΔΨ on vex-seq TR957) has many candidate features (e.g., 42 splicing-related features) and you must identify the minimal subset maximizing cross-validated PCC.

## When NOT to use

- The input is already a small, curated feature set where subset selection would overfit the CV metric
- The sample size is large enough for embedded feature-selection methods (e.g., regularized or deep models); the paper notes the small number of vex-seq samples is what motivated greedy selection over more powerful algorithms
- The target task is classification rather than ΔΨ regression evaluated by PCC, and no analogous validation metric is available per subset

## Inputs

- Feature matrix of 42 splicing-related features (splice-site, conservation, sequence-motif, regulatory-element, mRNA-structure) per vex-seq variant
- Observed ΔΨ values for the TR957 vex-seq training set (5th CAGI vex-seq challenge training data)
- Configured ε-SVR regressor (LIBSVM, RBF kernel) with grid-searched C and γ

## Outputs

- Ranked/ordered subset of selected features (eight features contributing most to prediction)
- Cross-validation PCC per feature-subset size, showing the peak at 8 features
- Final ε-SVR model retrained on the full TR957 set with the 8 selected features (writable as a LIBSVM model file)

## How to apply

Start from the precomputed feature matrix (42 splice-site, conservation, sequence-motif, regulatory-element, and mRNA-structure features per variant) and an ε-SVR with RBF kernel whose C and γ were grid-searched under 10-fold CV with folds grouped by gene to avoid within-gene leakage. At each selection round, evaluate every remaining feature by adding it to the current subset and re-running 10-fold cross-validation, keeping the single feature that yields the highest cross-validated Pearson correlation coefficient. Continue adding features one at a time and stop when the PCC peaks — for PredPSI-SVR on TR957, this occurred at eight features, with further additions decreasing performance. Retrain the final ε-SVR on the full training set using only the eight selected features and the optimal C/γ, then report the cross-validated predicted-versus-observed ΔΨ values for verification.

## Related tools

- **PredPSI-SVR** (Framework whose 42-feature matrix and ε-SVR (LIBSVM RBF) regression are the objects of greedy forward selection; selects the 8-feature final model) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Provides the ε-SVR implementation used to score each candidate feature subset via 10-fold CV PCC) — https://www.csie.ntu.edu.tw/~cjlin/libsvm

## Examples

```
cd ~/PredPSI-SVR/ && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- Cross-validated PCC should peak at the 8-feature subset and reproduce the reported value of 0.570 on TR957 within tolerance
- PCC should decrease (not increase) when a 9th feature is added, confirming the stopping criterion was applied correctly
- Selected subset size must be ≤ 42 (the full candidate feature count), with exactly one feature appended per selection round
- Final model retrained on the full TR957 set with the 8 selected features and grid-searched optimal C/γ must be written (e.g., as a LIBSVM model file) and its CV predictions-vs-observed ΔΨ table emitted
- Fold assignment must be grouped by gene so no within-gene leakage inflates the CV PCC used for selection

## Limitations

- Greedy selection is not exhaustive — it may miss feature combinations that a joint search would find, and it requires re-running 10-fold CV per candidate feature per round
- The 8-feature result is specific to the TR957 vex-seq training set and the ε-SVR/CV setup; other datasets or kernels may peak at different subset sizes
- PredPSI-SVR's underlying feature set omits branch point sequence features and does not effectively include pre-mRNA secondary structure features, so selection is limited to the 42 available features

## Evidence
