---
name: roc-auc-evaluation
description: Use when you have per-read posterior modification probabilities from
  a binary RNA-modification classifier (e.g. CHEUI Model 1 on IVT reads) and need
  to report accuracy, precision, recall, and ROC-AUC to validate or benchmark the
  model.
when_to_use_negative:
- Evaluating site-level stoichiometry or differential methylation (CHEUI Model 2 or
  CHEUI-diff outputs), where read-level AUC is not the appropriate metric
- Classifying 9-mer contexts with multiple modifiable bases (multiple A's or C's),
  which are excluded from the IVT test design
- Benchmarking tools that only output site-level calls without per-read probabilities,
  since ROC-AUC cannot be computed without score distributions
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3557
- http://edamontology.org/topic_3307
tools:
- name: CHEUI
  role: Two-stage deep learning (CNN) tool whose Model 1 read-level posterior probabilities
    for m6A and m5C are the scores being evaluated with accuracy, precision, recall,
    and AUC on IVT benchmarks
  repo: https://github.com/comprna/CHEUI
- name: nanopore direct RNA sequencing
  role: Source of the observed ionic-current signals processed into the 100-value
    + 100-distance feature vectors per 9-mer that the classifier scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:e5a8890167bdc0638f12a4a7ad2f3ab50a90ac303fa18f96a527b12904819035
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3557
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# roc-auc-evaluation

## Summary

Compute per-read classification metrics (accuracy, precision, recall, and AUC) from posterior modification probabilities produced by CHEUI-solo Model 1 on in-vitro transcript (IVT) test sets, to benchmark single-molecule m6A and m5C prediction performance against reported figures.

## When to use

Use when you have per-read posterior modification probabilities from a binary RNA-modification classifier (e.g. CHEUI Model 1 on IVT reads) and need to report accuracy, precision, recall, and ROC-AUC to validate or benchmark the model.

## When NOT to use

- Evaluating site-level stoichiometry or differential methylation (CHEUI Model 2 or CHEUI-diff outputs), where read-level AUC is not the appropriate metric
- Classifying 9-mer contexts with multiple modifiable bases (multiple A's or C's), which are excluded from the IVT test design
- Benchmarking tools that only output site-level calls without per-read probabilities, since ROC-AUC cannot be computed without score distributions

## Inputs

- Preprocessed CHEUI read-signal feature matrices (100 observed signal values + 100 distances to Nanopolish 5-mer model per 9-mer) for IVT test 1 and IVT test 2
- Nanopolish eventalign output preprocessed with CHEUI_preprocess_m6A.py (m6A) or CHEUI_preprocess_m5C.py (m5C)
- Trained CHEUI-solo Model 1 Keras/TensorFlow model files (CHEUI_m6A_model1.h5, CHEUI_m5C_model1.h5)
- Binary ground-truth labels (modified vs unmodified) for each IVT test read

## Outputs

- Per-read posterior modification probability table (read ID, 9-mer context, probability)
- Metrics table (accuracy, precision, recall, AUC) per modification and test set, e.g. cheui_model1_ivt_test_metrics.csv

## How to apply

Load the preprocessed read-signal inputs (100 median-vectorized observed signal values plus 100 absolute distances from the Nanopolish 5-mer model per 9-mer) for IVT test 1 and IVT test 2, restricting to 9-mers with exactly one A (m6A) or one C (m5C) at the center. Apply the trained CHEUI-solo Model 1 CNNs (Keras/TensorFlow, e.g. CHEUI_m6A_model1.h5 and CHEUI_m5C_model1.h5) separately to each test set to obtain a posterior modification probability per read. Compute accuracy as (TN+TP)/(TN+TP+FN+FP), precision as TP/(TP+FP), and recall as TP/(TP+FN) for each modification and test set, and derive the AUC from the per-read probability distributions. Compare the computed metrics against the reported values — m6A accuracy 0.835 (test 1) / 0.777 (test 2), m5C accuracy 0.793/0.741; m6A AUC 0.910/0.856, m5C AUC 0.879/0.827 — and write the result table (e.g. cheui_model1_ivt_test_metrics.csv).

## Related tools

- **CHEUI** (Two-stage deep learning (CNN) tool whose Model 1 read-level posterior probabilities for m6A and m5C are the scores being evaluated with accuracy, precision, recall, and AUC on IVT benchmarks) — https://github.com/comprna/CHEUI
- **nanopore direct RNA sequencing** (Source of the observed ionic-current signals processed into the 100-value + 100-distance feature vectors per 9-mer that the classifier scores)

## Examples

```
python scripts/CHEUI_predict_model1.py --input ivt_test1_features.txt --DL_model CHEUI_trained_models/CHEUI_m6A_model1.h5 --output ivt_test1_m6A_probs.csv
```

## Evaluation signals

- m6A accuracy/precision/recall/AUC should reproduce ~0.835/0.820/0.853/0.910 on IVT set 1 and ~0.777/0.844/0.791/0.856 on IVT set 2
- m5C accuracy and AUC should fall within a few percent of 0.793/0.741 (test 1) and 0.741/0.827 (test 2)
- Metrics table must contain four values (accuracy, precision, recall, AUC) for each combination of modification (m6A, m5C) and test set (1, 2)
- Feature vectors per read must contain 200 values: 100 median-vectorized observed signals plus 100 absolute distances to the Nanopolish 5-mer model
- AUC computed from per-read probability distributions should exceed accuracy for all four modification/test-set combinations, as in the reported figures

## Limitations

- k-mer generalization degrades performance: metrics drop for IVT set 2 (contexts not seen during training) relative to IVT set 1 (contexts seen during training)
- Read-level discrimination can fail when the nanopore signal difference between modified and non-modified reads is comparable to variation among non-modified reads
- Modifications closer than 5 nt to each other show signal interference, complicating single-read prediction (exacerbated in densely modified rRNA)

## Evidence

- [results] CHEUI-solo Model 1 achieved m6A metrics (accuracy, precision, recall, AUC) of 0.835/0.820/0.853/0.910 for IVT set 1 and 0.777/0.844/0.791/0.856 for IVT set 2.: [evidence span withheld — anchor status: not_found]
- [methods] Load the preprocessed read-signal inputs (100 median-vectorized observed signal values plus 100 absolute distances from the Nanopolish 5 mer model per 9 mer): [evidence span withheld — anchor status: not_found]
- [intro] CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs): "CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs)"
- [abstract] CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [readme] CHEUI model 1 (read level predictions) and model 2 (site level predictions) use different predictive models for m6A and m5C: [evidence span withheld — anchor status: external_doc]
