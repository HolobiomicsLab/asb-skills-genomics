---
name: gene-grouped-10-fold-cross-validation
description: Use when training or tuning a regression model (e.g., ε-SVR with RBF
  kernel) on variant-level ΔΨ data where multiple variants come from the same gene
  and naive random folds would leak information between train and validation splits.
when_to_use_negative:
- When each sample comes from a distinct gene, so no leakage risk exists and simple
  random k-fold CV suffices
- When the goal is an independent blind-test evaluation rather than hyper-parameter
  tuning or feature selection on a training set
- When sample size is large enough for deep learning approaches — the article notes
  the small sample number (TR957) constrained the method choice
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PredPSI-SVR
  role: 'Model under evaluation: ε-SVR predicting ΔΨ, whose hyper-parameters and feature
    set are optimized by the grouped 10-fold CV'
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Implements ε-SVR with RBF kernel; C and γ are grid-searched within the grouped
    10-fold CV and the final model is saved as a LIBSVM model file
  repo: https://www.csie.ntu.edu.tw/~cjlin/libsvm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:8b8d33957760576187a09d369fb71e03a37621ff5a646d915edfb4da1b359f46
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - PredPSI-SVR
  - Python 3
  - libsvm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Gene-grouped 10-fold cross-validation

## Summary

A cross-validation scheme for the PredPSI-SVR ΔΨ-prediction model in which 10-fold CV folds are constructed by grouping vex-seq variants by gene, preventing within-gene leakage while grid-searching ε-SVR hyperparameters and estimating PCC on the TR957 training set.

## When to use

Use when training or tuning a regression model (e.g., ε-SVR with RBF kernel) on variant-level ΔΨ data where multiple variants come from the same gene and naive random folds would leak information between train and validation splits.

## When NOT to use

- When each sample comes from a distinct gene, so no leakage risk exists and simple random k-fold CV suffices
- When the goal is an independent blind-test evaluation rather than hyper-parameter tuning or feature selection on a training set
- When sample size is large enough for deep learning approaches — the article notes the small sample number (TR957) constrained the method choice

## Inputs

- TR957 vex-seq training set (CAGI 5) with observed ΔΨ values per variant
- Precomputed matrix of 42 splicing-related features per variant (splice-site, conservation, sequence-motif, regulatory-element, mRNA-structure features)
- Gene/parent-gene identifier for each variant (for fold grouping)

## Outputs

- 10-fold CV predicted-versus-observed ΔΨ prediction table
- Optimal ε-SVR hyperparameters C and γ (RBF kernel)
- Feature-selection trace (PCC per number of selected features, peaking at 8 features, PCC 0.570)
- Trained ε-SVR model written to a LIBSVM model file

## How to apply

Group the TR957 vex-seq training variants by gene so that all variants from one gene fall in the same fold, avoiding within-gene leakage between training and validation splits. Under each fold combination, grid-search the ε-SVR regularization parameter C and RBF kernel width γ with LIBSVM and compute the Pearson correlation coefficient (PCC) of predicted versus observed ΔΨ. Use the same grouped 10-fold CV to drive greedy forward feature selection: at each round add the single remaining feature (from the 42 splicing-related features) that maximizes the CV PCC, stopping when PCC peaks — here at eight features with PCC 0.570. Retrain the final ε-SVR on the full TR957 set with the selected features and optimal C/γ before blind-test application.

## Related tools

- **PredPSI-SVR** (Model under evaluation: ε-SVR predicting ΔΨ, whose hyper-parameters and feature set are optimized by the grouped 10-fold CV) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Implements ε-SVR with RBF kernel; C and γ are grid-searched within the grouped 10-fold CV and the final model is saved as a LIBSVM model file) — https://www.csie.ntu.edu.tw/~cjlin/libsvm

## Examples

```
python train_svr.py --data TR957_features.tsv --groups gene_id --model libsvm --kernel rbf --cv 10 --grid-search C,gamma --feature-selection greedy
```

## Evaluation signals

- The 10-fold CV PCC of the 8-feature ε-SVR model reproduces the reported peak value of 0.570 within tolerance
- PCC increases monotonically during greedy forward selection up to 8 features and decreases with further additions
- No gene appears in more than one CV fold (within-gene leakage check)
- Cross-validation predictions versus observed ΔΨ form the expected output prediction table
- Retrained model with selected features and optimal C/γ yields PCC ~0.570 on training CV and 0.566 on the independent blind test

## Limitations

- Small sample size prevented the use of more powerful algorithms such as deep learning
- Performance depends on wild-type Ψ as input: without experimental Ψ, PCCs on the training and test sets dropped by about 0.1
- Branch point sequence and pre-mRNA secondary structure features were not effectively included due to limited samples

## Evidence

- [methods] Set up ε-SVR with LIBSVM using the RBF kernel and grid-search the regularization parameter C and the kernel width γ under 10-fold cross-validation in which folds are constructed by grouping variants by gene to avoid within-gene leakage.: [evidence span withheld — anchor status: not_found]
