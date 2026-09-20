---
name: delta-psi-prediction-performance-evaluation
description: Use when a trained ΔΨ prediction model (e.g. ε-SVR on vex-seq variants)
  must be assessed via 10-fold cross-validation PCC, or when benchmarking against
  PredPSI-SVR's reported 0.570 CV / 0.566 blind-test performance.
when_to_use_negative:
- Input is already an annotated, model-ready feature matrix and only a delta-PSI prediction
  is needed (use the PredPSI-SVR prediction workflow, not evaluation).
- The dataset has too few samples with extreme ΔΨ values removed (fewer than the 53
  mutations retained after removing ΔΨ with absolute values less than two times the
  standard deviation), making CV PCC unstable.
- The goal is splice-site or branch-point feature prediction for a single exon, not
  variant-level ΔΨ regression evaluation — PredPSI-SVR does not include branch point
  sequence features.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: PredPSI-SVR
  role: ε-SVR model whose 10-fold cross-validation PCC on ΔΨ predictions is evaluated;
    provides feature extraction and trained model for benchmarking
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Implements the ε-SVR with RBF kernel used for grid-searching C and γ during
    10-fold cross-validation
  repo: https://www.csie.ntu.edu.tw/~cjlin/libsvm
- name: SPANR
  role: Competing method whose SPIDEX ΔΨ predictions serve as baseline comparison
    for ΔΨ prediction accuracy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:baa1e4c371b0197e85aa980b90165086c02eb7c22b5d74be1f1eb26216722ea8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - PredPSI-SVR
  - Python 3
  - SPANR
  - libsvm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# delta-psi-prediction-performance-evaluation

## Summary

Evaluates support-vector-regression models predicting variant-induced ΔΨ (change of percent spliced in) by computing Pearson correlation coefficient (PCC) under 10-fold cross-validation on the CAGI 5 vex-seq TR957 training set and comparing against the reported PredPSI-SVR benchmark of PCC 0.570 (CV) and 0.566 (blind test).

## When to use

Use when a trained ΔΨ prediction model (e.g. ε-SVR on vex-seq variants) must be assessed via 10-fold cross-validation PCC, or when benchmarking against PredPSI-SVR's reported 0.570 CV / 0.566 blind-test performance.

## When NOT to use

- Input is already an annotated, model-ready feature matrix and only a delta-PSI prediction is needed (use the PredPSI-SVR prediction workflow, not evaluation).
- The dataset has too few samples with extreme ΔΨ values removed (fewer than the 53 mutations retained after removing ΔΨ with absolute values less than two times the standard deviation), making CV PCC unstable.
- The goal is splice-site or branch-point feature prediction for a single exon, not variant-level ΔΨ regression evaluation — PredPSI-SVR does not include branch point sequence features.

## Inputs

- TR957 vex-seq training dataset (CAGI 5 vex-seq challenge) with observed ΔΨ values
- Feature matrix of 42 splicing-related features per variant
- Cross-validation predicted-versus-observed ΔΨ prediction table
- Blind vex-seq test set predictions (for independent-test PCC)

## Outputs

- 10-fold cross-validation PCC of predicted versus observed ΔΨ
- Prediction table of CV predicted-versus-observed ΔΨ values
- PCC-per-feature-count curve from greedy forward selection (peak at 8 features)
- Trained LIBSVM model file for the final ε-SVR model

## How to apply

Load the TR957 vex-seq training set with the precomputed matrix of 42 splicing-related features (splice-site, conservation, sequence-motif, regulatory-element, and mRNA-structure features). Set up ε-SVR with LIBSVM using the RBF kernel and grid-search C and γ under 10-fold cross-validation, constructing folds by grouping variants by gene to avoid within-gene leakage. Evaluate the greedy forward feature selection loop, where at each round the single feature maximizing 10-fold CV PCC is added; performance peaked at eight features (PCC 0.570) with further additions decreasing performance. Compute the PCC of cross-validation predicted-versus-observed ΔΨ values and confirm the 8-feature model reproduces 0.570 within tolerance; additionally compare against the blind-test PCC of 0.566 and SPANR's SPIDEX ΔΨ baseline, where PredPSI-SVR increased PCC from 0.566 to 0.665 in the challenge discussion.

## Related tools

- **PredPSI-SVR** (ε-SVR model whose 10-fold cross-validation PCC on ΔΨ predictions is evaluated; provides feature extraction and trained model for benchmarking) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Implements the ε-SVR with RBF kernel used for grid-searching C and γ during 10-fold cross-validation) — https://www.csie.ntu.edu.tw/~cjlin/libsvm
- **SPANR** (Competing method whose SPIDEX ΔΨ predictions serve as baseline comparison for ΔΨ prediction accuracy)

## Examples

```
cd ~/PredPSI-SVR/ && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- 10-fold cross-validation PCC of predicted vs observed ΔΨ reproduces the reported peak of 0.570 within tolerance for the 8-feature model.
- Blind-test PCC matches the reported 0.566 (ranked 2nd in the CAGI 5 vex-seq challenge).
- Greedy forward selection curve shows PCC increasing up to 8 features and decreasing with further additions.
- CV folds are grouped by gene, so no within-gene leakage inflates PCC.
- Model outperforms the SPANR/SPIDEX ΔΨ baseline in cross-validation.

## Limitations

- Without experimental wild-type Ψ of exons as input, PCCs on the training set and test set dropped by about 0.1.
- The small number of samples prevented use of more powerful algorithms like deep learning.
- PredPSI-SVR does not include features of branch point sequences and pre-mRNA secondary structure effectively due to the limited sample size.
- Only Human BodyMap Project 2.0 RNA-seq data was used, and tissue-specific Ψ was not incorporated.

## Evidence
