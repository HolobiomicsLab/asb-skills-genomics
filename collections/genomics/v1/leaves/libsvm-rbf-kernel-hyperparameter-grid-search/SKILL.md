---
name: libsvm-rbf-kernel-hyperparameter-grid-search
description: Use when training an ε-SVR model on a small feature matrix (e.g., tens
  of samples, dozens of features such as the 42 splicing-related vex-seq features)
  and you must select C and RBF γ by cross-validated correlation rather than default
  parameters.
when_to_use_negative:
- Input is already a trained model file or a reduced feature set — the grid search
  is meant for raw candidate feature matrices before feature selection.
- The dataset is large enough for more expressive models (e.g., deep learning); the
  authors note the small vex-seq sample size is what justified SVR, and PredPSI-SVR
  omits branch point and pre-mRNA secondary structure features for this reason.
- The task requires predictions on variants more than 200 bp from splice sites or
  in intergenic regions — PredPSI-SVR filters these out before modeling.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: PredPSI-SVR
  role: 'Wraps the ε-SVR pipeline: feature extraction from exon/flanking sequence,
    greedy feature selection, and RBF-kernel SVR training with grid-searched C/γ'
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Provides the ε-SVR implementation with RBF kernel (libsvm-3.23) used for grid
    search and final model training; bundled in PredPSI-SVR/tools
  repo: https://www.csie.ntu.edu.tw/~cjlin/libsvm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:65ba3c2a57b3965c7d8836b98178a4c3b1ad27b4a6bfa81d01537fb65464a154
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - PredPSI-SVR
  - Python 3
  - libsvm
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# libsvm-rbf-kernel-hyperparameter-grid-search

## Summary

Hyperparameter optimization strategy for ε-support vector regression using LIBSVM's RBF kernel, in which the regularization parameter C and kernel width γ are grid-searched under 10-fold cross-validation, used by PredPSI-SVR to predict ΔΨ from 42 splicing-related vex-seq features and reach a cross-validation PCC of 0.570.

## When to use

Use when training an ε-SVR model on a small feature matrix (e.g., tens of samples, dozens of features such as the 42 splicing-related vex-seq features) and you must select C and RBF γ by cross-validated correlation rather than default parameters.

## When NOT to use

- Input is already a trained model file or a reduced feature set — the grid search is meant for raw candidate feature matrices before feature selection.
- The dataset is large enough for more expressive models (e.g., deep learning); the authors note the small vex-seq sample size is what justified SVR, and PredPSI-SVR omits branch point and pre-mRNA secondary structure features for this reason.
- The task requires predictions on variants more than 200 bp from splice sites or in intergenic regions — PredPSI-SVR filters these out before modeling.

## Inputs

- Feature matrix of 42 splicing-related features per variant (splice-site, conservation, sequence-motif, regulatory-element, mRNA-structure), e.g., the TR957 vex-seq training set
- Observed ΔΨ (delta percent-spliced-in) values for each training variant
- Gene/variant grouping labels for leakage-free CV fold construction

## Outputs

- Optimal C and γ hyperparameter values selected by 10-fold cross-validated PCC
- Trained LIBSVM ε-SVR model written to a LIBSVM model file
- 10-fold cross-validation predicted-versus-observed ΔΨ prediction table with PCC (0.570 at peak)
- Ordered list of greedily selected features (eight features for PredPSI-SVR)

## How to apply

Build the ε-SVR with LIBSVM using the RBF kernel, then grid-search the regularization parameter C and kernel width γ, scoring each parameter pair by the Pearson correlation coefficient (PCC) under 10-fold cross-validation on the training set (e.g., TR957 vex-seq variants). Construct CV folds by grouping variants by gene so that no gene contributes samples to both train and validation folds, preventing within-gene leakage. After fixing the optimal C/γ, run greedy forward feature selection over the 42 candidate features: at each round add the single remaining feature that maximizes the 10-fold CV PCC, and stop when the PCC peaks — for PredPSI-SVR this peaked at 0.570 with eight features. Retrain the final ε-SVR on the full training set with the selected features and write the model to a LIBSVM model file plus a predicted-versus-observed ΔΨ table.

## Related tools

- **PredPSI-SVR** (Wraps the ε-SVR pipeline: feature extraction from exon/flanking sequence, greedy feature selection, and RBF-kernel SVR training with grid-searched C/γ) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Provides the ε-SVR implementation with RBF kernel (libsvm-3.23) used for grid search and final model training; bundled in PredPSI-SVR/tools) — https://www.csie.ntu.edu.tw/~cjlin/libsvm

## Examples

```
cd ~/PredPSI-SVR/ && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- 10-fold CV PCC of the final 8-feature model reproduces the reported peak of 0.570 within tolerance.
- The CV PCC curve over greedy forward selection peaks at exactly eight features, with PCC decreasing when further features are added.
- Fold assignment check: no gene has variants in both the training and validation partitions of any CV fold.
- Grid search reports the chosen C/γ as the pair maximizing CV PCC, not defaults.
- Prediction table pairs every cross-validation prediction with its observed ΔΨ so the PCC can be recomputed independently.

## Limitations

- Method requires wild-type Ψ of exons as input; without experimental Ψ, PCCs on training and test sets dropped by about 0.1.
- Small sample size (TR957) limits model capacity and prevented use of deep learning; branch point and pre-mRNA secondary structure features are not effectively included.
- Variants in intergenic regions or more than 200 bp from splice sites are filtered out before prediction, so fewer outputs than input VCF entries can occur.

## Evidence
