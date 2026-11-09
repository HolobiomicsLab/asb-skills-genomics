---
name: binary-classifier-metric-computation
description: Use when a per-read binary modification classifier (e.g. CHEUI-solo Model
  1 CNN outputs on IVT set 1/set 2 9-mer signals) must be scored with accuracy, precision,
  recall, and AUC and checked against published values.
when_to_use_negative:
- Input is already an aggregated site-level stoichiometry table rather than per-read
  probabilities — site-level metrics require CHEUI Model 2 logic instead
- The goal is differential methylation between two conditions, which is handled by
  CHEUI-diff rather than single-classifier evaluation
- The test 9-mers overlap training contexts, so the exercise would not measure k-mer
  generalization (use held-out IVT set 2 for that)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CHEUI
  role: Provides the trained Model 1 CNNs that yield per-read m6A/m5C posterior probabilities
    to be scored; models loaded via --DL_model (e.g. CHEUI_m6A_model1.h5)
  repo: https://github.com/comprna/CHEUI
- name: nanopore direct RNA sequencing
  role: Source of the ionic-current signals from which the 100-value observed signal
    vectors per 9-mer are derived
- name: Nanopolish
  role: eventalign produces the expected 5-mer signal model used to compute the 100
    absolute-distance features per 9-mer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:3fd35b6289f79a46075ca5e8a88047d5c100ecfce23454f490e7dd7eb9c15c84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Nanopolish
  - nanopore direct RNA sequencing
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# binary-classifier-metric-computation

## Summary

Computes and validates standard binary classification metrics (accuracy, precision, recall, AUC) for CHEUI-solo Model 1 read-level m6A and m5C predictions on IVT test sets, enabling direct comparison against the published benchmark figures.

## When to use

Use when a per-read binary modification classifier (e.g. CHEUI-solo Model 1 CNN outputs on IVT set 1/set 2 9-mer signals) must be scored with accuracy, precision, recall, and AUC and checked against published values.

## When NOT to use

- Input is already an aggregated site-level stoichiometry table rather than per-read probabilities — site-level metrics require CHEUI Model 2 logic instead
- The goal is differential methylation between two conditions, which is handled by CHEUI-diff rather than single-classifier evaluation
- The test 9-mers overlap training contexts, so the exercise would not measure k-mer generalization (use held-out IVT set 2 for that)

## Inputs

- Preprocessed read-signal feature matrices for IVT test 1 and IVT test 2 (100 observed signal values + 100 Nanopolish 5-mer-model distances per 9-mer)
- Trained CHEUI-solo Model 1 .h5 CNN weights for m6A and m5C (Keras/TensorFlow)
- Set of 9-mer sites with exactly one central A (m6A) or one central C (m5C)

## Outputs

- Per-read modification posterior probabilities for m6A and m5C on both IVT test sets
- Metrics table cheui_model1_ivt_test_metrics.csv with accuracy, precision, recall, and AUC per modification per test set

## How to apply

Load the preprocessed read-signal inputs, each read represented as 100 median-vectorized observed signal values plus 100 absolute distances from the Nanopolish 5-mer model per 9-mer, restricted to 9-mers with only one A (m6A) or one C (m5C) at the center. Apply the trained CHEUI-solo Model 1 Keras/TensorFlow CNNs (e.g. CHEUI_m6A_model1.h5 and CHEUI_m5C_model1.h5) separately to IVT test 1 (sensor generalization) and IVT test 2 (k-mer generalization) to obtain a posterior modification probability per read. Derive TN/TP/FN/FP counts from the per-read predictions and compute accuracy as (TN+TP)/(TN+TP+FN+FP), precision as TP/(TP+FP), and recall as TP/(TP+FN) for each modification and test set. Compute the AUC per modification and test set from the per-read probability distributions. Write the results table to cheui_model1_ivt_test_metrics.csv and judge correctness by agreement with reported values (m6A accuracy 0.835/0.777, AUC 0.910/0.856; m5C accuracy 0.793/0.741, AUC 0.879/0.827 for test 1/test 2), within a few percent.

## Related tools

- **CHEUI** (Provides the trained Model 1 CNNs that yield per-read m6A/m5C posterior probabilities to be scored; models loaded via --DL_model (e.g. CHEUI_m6A_model1.h5)) — https://github.com/comprna/CHEUI
- **nanopore direct RNA sequencing** (Source of the ionic-current signals from which the 100-value observed signal vectors per 9-mer are derived)
- **Nanopolish** (eventalign produces the expected 5-mer signal model used to compute the 100 absolute-distance features per 9-mer)

## Examples

```
accuracy = (TN+TP)/(TN+TP+FN+FP); precision = TP/(TP+FP); recall = TP/(TP+FN); metrics.to_csv('cheui_model1_ivt_test_metrics.csv', index=False)
```

## Evaluation signals

- Reproduced m6A accuracy/precision/recall/AUC fall within a few percent of 0.835/0.820/0.853/0.910 (IVT set 1) and 0.777/0.844/0.791/0.856 (IVT set 2)
- Reproduced m5C accuracy and AUC fall within a few percent of 0.793/0.879 (test 1) and 0.741/0.827 (test 2)
- All per-read probabilities lie in [0, 1] and AUC is computed from the full probability distributions, not hard labels
- Confusion-matrix counts satisfy accuracy = (TN+TP)/(TN+TP+FN+FP) with precision TP/(TP+FP) and recall TP/(TP+FN)
- Results are written to cheui_model1_ivt_test_metrics.csv with one row per modification per test set

## Limitations

- Read-level performance drops on k-mer contexts unseen during training (IVT set 2: m6A accuracy 0.777 vs 0.835), reflecting limited k-mer generalization
- Nanopore signal variability means differences between modified and non-modified reads are often comparable to variation among non-modified reads, capping achievable read-level discrimination
- Very strict cut-offs used on cell transcriptome sites to keep expected FDR low reduce CHEUI sensitivity, so threshold choice strongly affects downstream site-level performance

## Evidence

- [abstract] CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [other] CHEUI-solo Model 1 achieved m6A metrics (accuracy, precision, recall, AUC) of 0.835/0.820/0.853/0.910 for IVT set 1 and 0.777/0.844/0.791/0.856 for IVT set 2.: [evidence span withheld — anchor status: not_found]
- [readme] CHEUI uses a two-stage deep learning method to detect m6A and m5C transcriptome-wide at single-read and single-site resolution: [evidence span withheld — anchor status: external_doc]
- [readme] for m6A: ../CHEUI_trained_models/CHEUI_m6A_model1.h5 and ../CHEUI_trained_models/CHEUI_m6A_model2.h5: "for m6A: 
        ../CHEUI_trained_models/CHEUI_m6A_model1.h5 and ../CHEUI_trained_models/CHEUI_m6A_model2.h5"
- [discussion] Improvements in the prediction models and in the RNA sequencing chemistry can potentially facilitate the identification of m6A and m5C at higher sensitivity.: "Improvements in the prediction models and in the RNA sequencing chemistry can potentially facilitate the identification of m6A and m5C at higher sensitivity."
