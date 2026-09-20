---
name: cross-dataset-model-generalization-testing
description: Use when a trained model artifact (e.g., scripted_m21.pth) must be evaluated
  on held-out BAMs from organisms/chemistry differing from its training set, to quantify
  generalization via AUC/F1 before deployment.
when_to_use_negative:
- When the goal is site-specific Z/A calling within hybrid reads — the k21-full-ZA
  model classifies entire reads as dZ-modified or native; use the k11-mixed-AZ model
  with -k 11 instead
- When reads lack kinetic tags or fail the Median_PW/Median_IPD 0.3-99th-percentile
  and 3-pass filters — kinetics-based generalization testing is invalid on unfiltered
  or tag-less BAMs
- When classifying hybrid reads with diluted kinetic signatures — the read classifier
  is optimized for fully substituted dZ-genomes and may leave such reads unclassified
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Z-Calling
  role: Provides filter_bam.py preprocessing, the z-calling-base MLP module used to
    score held-out BAMs at -k 21, and the read-level aggregation used to compute AUC/F1
    per dataset
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PyTorch
  role: Trains the MLP classifier on E. coli dZ-DNA vs canonical reads for each candidate
    k-mer size and exports the frozen TorchScript artifact (scripted_m21.pth) evaluated
    on cross-datasets
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio CCS
  role: Source of HiFi reads with preserved PW/IPD kinetics (via --keep-kinetics)
    across Revio and Sequel II datasets used as train/test partitions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:9b5d098eadc9a2f4499c1ee593ddca3c06a2d1fda7ba13a07babd981486e41d9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PacBio CCS
  - PyTorch
  - Z-Calling
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-dataset-model-generalization-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Test whether a dZ-DNA/canonical read-classification model (e.g., the k21-full-ZA MLP trained on E. coli Revio reads) generalizes to held-out datasets from different organisms and sequencing chemistries (fruit fly Revio and Sequel II), by running the frozen model and computing AUC and F1 per test set.

## When to use

Use when a trained model artifact (e.g., scripted_m21.pth) must be evaluated on held-out BAMs from organisms/chemistry differing from its training set, to quantify generalization via AUC/F1 before deployment.

## When NOT to use

- When the goal is site-specific Z/A calling within hybrid reads — the k21-full-ZA model classifies entire reads as dZ-modified or native; use the k11-mixed-AZ model with -k 11 instead
- When reads lack kinetic tags or fail the Median_PW/Median_IPD 0.3-99th-percentile and 3-pass filters — kinetics-based generalization testing is invalid on unfiltered or tag-less BAMs
- When classifying hybrid reads with diluted kinetic signatures — the read classifier is optimized for fully substituted dZ-genomes and may leave such reads unclassified

## Inputs

- Held-out filtered PacBio HiFi/CCS BAM files with kinetic tags (fi, ri, fp, rp) from non-training datasets (fruit fly Revio, fruit fly Sequel II, E. coli Revio)
- Frozen TorchScript model artifact model/k21-full-ZA/scripted_m21.pth
- Training-set filtered BAMs of dZ-DNA (full-Z) and canonical (full-dA) E. coli Revio reads

## Outputs

- Output BAM with per-base Z probabilities in MM/ML and ZP tags per held-out dataset
- Read-level dZ-DNA vs canonical predictions per test set
- Evaluation TSV with per-dataset AUC and F1 scores
- k-mer selection table/plots of AUC and CPU training/calling time versus k (7-25)

## How to apply

First run the preprocessing filter (filter_bam.py) on both training and held-out BAMs, keeping reads with kinetic tags (fi, ri, fp, rp), discarding reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile, and requiring fn >= 3 and rn >= 3 passes. Train the PyTorch MLP on dZ-DNA (full-Z) versus canonical (full-dA) E. coli Revio reads after selecting the feature k-mer size (21 bp was chosen as the balance of model strength and CPU cost across odd k in 7-25). Freeze the model as a TorchScript artifact (model/k21-full-ZA/scripted_m21.pth) so no retraining or per-dataset tuning occurs, then run build/z-calling-base -k 21 on the held-out fruit fly (Revio and Sequel II) and E. coli (Revio) test BAMs, writing per-base Z probabilities into MM/ML and ZP tags. Aggregate read-level predictions per test set and compute AUC and F1 for dZ-DNA versus canonical classification per organism/chemistry. The rationale is that generalization across datasets demonstrates the kinetic (PW/IPD) signatures are species- and run-independent rather than overfit to the training E. coli data.

## Related tools

- **Z-Calling** (Provides filter_bam.py preprocessing, the z-calling-base MLP module used to score held-out BAMs at -k 21, and the read-level aggregation used to compute AUC/F1 per dataset) — https://github.com/xiaochuanle/Z-Calling
- **PyTorch** (Trains the MLP classifier on E. coli dZ-DNA vs canonical reads for each candidate k-mer size and exports the frozen TorchScript artifact (scripted_m21.pth) evaluated on cross-datasets) — https://github.com/xiaochuanle/Z-Calling
- **PacBio CCS** (Source of HiFi reads with preserved PW/IPD kinetics (via --keep-kinetics) across Revio and Sequel II datasets used as train/test partitions)

## Examples

```
build/z-calling-base -k 21 -t 16 FilteredTestBAM.bam model/k21-full-ZA/scripted_m21.pth OutputBAM.bam
```

## Evaluation signals

- Evaluation table exists as a TSV and per-dataset AUC values fall within the reported 0.94-0.96 range and F1 scores within 0.85-0.92 for the Z base calling module
- The specific k21-full-ZA cross-dataset evaluation yields AUC 0.9646-0.9796 and F1 0.9045-0.9384 across fruit fly (Revio and Sequel II) and E. coli (Revio) test sets
- The model artifact used for all test sets is the same frozen scripted_m21.pth invoked with -k 21 — no per-dataset retraining or threshold refitting
- Input BAMs pass filter_bam.py criteria: kinetic tags present, Median_PW/Median_IPD in [0.3, 99th percentile], fn >= 3 and rn >= 3 passes
- k-mer sweep confirms k=21 was selected by jointly inspecting AUC and CPU training/calling time versus k for odd k in 7-25

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures
- The subtle Z-induced PW shift (0.5-1.5-fold) overlaps with canonical adenine variance, limiting hybrid-sample quantification
- Generalization was only demonstrated on fully substituted dZ-DNA versus canonical datasets; no ground-truth hybrid training data with defined Z-positions existed for testing intermediate cases

## Evidence

- [readme] readme performance range: "Across multiple datasets, the Z base calling module achieves **AUCs of 0.94–0.96** and **F1 scores of 0.85–0.92**."
