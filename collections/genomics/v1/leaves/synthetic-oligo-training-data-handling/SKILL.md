---
name: synthetic-oligo-training-data-handling
description: Use when you need labeled m6A/unm6A training examples for base-called-feature
  classifiers from ONT direct RNA data, e.g. training or reproducing EpiNano SVMs
  on synthetic RRACH k-mers where ground-truth modification status is known by design.
when_to_use_negative:
- In vivo datasets without ground-truth modification labels — the article recommends
  pairing EpiNano with a matched control (WT vs KO/KD or demethylase-treated) instead
  of synthetic-oligo supervised training
- Per-read (single-molecule) modification calling — the algorithm aggregates information
  from all reads mapping to a site and does not predict modifications in individual
  RNA molecules
- Analyses requiring manipulation of raw current intensity (squiggle realignment)
  — EpiNano works on base-called features without touching raw signal data
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: EpiNano
  role: Extracts per-k-mer base-calling 'error' features (mismatch, insertion, deletion,
    base quality, current intensity in v1.0) from ONT direct RNA reads to build the
    training tables for the m6A SVMs
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the synthetic-oligo reads whose base-calling
    errors encode m6A modification status
- name: scikit-learn
  role: Trains the individual per-feature and combined-feature binary SVM classifiers
    on the per-site feature tables and evaluates them on held-out labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:90a0672d5ba36f960389d23454720b5f85910aac00c93561539bd4e924baffce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# synthetic-oligo-training-data-handling

## Summary

Prepares and uses in vitro synthetic RNA oligos with known m6A-modified and unmodified RRACH sites as ground-truth training data for EpiNano's SVM-based m6A classification on Oxford Nanopore direct RNA reads, exploiting base-called 'error' features (mismatch, deletion, base quality) rather than raw current signals.

## When to use

Use when you need labeled m6A/unm6A training examples for base-called-feature classifiers from ONT direct RNA data, e.g. training or reproducing EpiNano SVMs on synthetic RRACH k-mers where ground-truth modification status is known by design.

## When NOT to use

- In vivo datasets without ground-truth modification labels — the article recommends pairing EpiNano with a matched control (WT vs KO/KD or demethylase-treated) instead of synthetic-oligo supervised training
- Per-read (single-molecule) modification calling — the algorithm aggregates information from all reads mapping to a site and does not predict modifications in individual RNA molecules
- Analyses requiring manipulation of raw current intensity (squiggle realignment) — EpiNano works on base-called features without touching raw signal data

## Inputs

- Synthetic m6A-modified direct RNA reads containing RRACH 5-mers with known modification sites (ONT data)
- Matching synthetic unmodified control direct RNA reads (A at RRACH positions)
- Base-called FASTQ/BAM output from ONT base-caller (e.g. Albacore 2.1.7 or Guppy 3.1.5)
- Albacore FAST5 files if current intensity features are required (v1.0 only)

## Outputs

- Per-site feature tables (mismatch frequency, deletion frequency, base quality, optionally current intensity) for modified vs unmodified RRACH sites
- Trained per-feature and combined-feature binary SVM models
- Classification accuracy results table (e.g. svm_accuracy_results.csv) with per-feature, combined-feature, and current-intensity accuracies

## How to apply

Process synthetic direct RNA reads containing m6A at known RRACH positions and matched unmodified controls through the ONT base-caller, then run the EpiNano pipeline to extract per-site features at position 0 and neighboring positions of the target 5-mer: mismatch frequency, deletion frequency, base quality, and current intensity (current intensity is optional — v1.1+ of EpiNano dropped it because it was not used to train the final models). Build per-site feature tables for the modified and unmodified sites. Train individual binary SVM classifiers in scikit-learn, one per feature, plus a combined-feature SVM using position-0 and neighbor features together. The rationale is that m6A manifests as systematic base-calling 'errors' and decreased quality scores in base-callers that do not model modifications, so synthetic oligos with known labels provide clean supervision. Evaluate each SVM on held-out per-site labels and record accuracies, expecting per-feature accuracies of 70–86%, combined-feature accuracy of 88–91%, and current-intensity-only accuracy of only 43–65%.

## Related tools

- **EpiNano** (Extracts per-k-mer base-calling 'error' features (mismatch, insertion, deletion, base quality, current intensity in v1.0) from ONT direct RNA reads to build the training tables for the m6A SVMs) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the synthetic-oligo reads whose base-calling errors encode m6A modification status)
- **scikit-learn** (Trains the individual per-feature and combined-feature binary SVM classifiers on the per-site feature tables and evaluates them on held-out labels)

## Examples

```
from sklearn.svm import SVC; clf = SVC(); clf.fit(site_features_train, y_train)
```

## Evaluation signals

- Per-feature SVM accuracies fall in the 70–86% range on held-out synthetic per-site labels
- Combined-feature (base quality + mismatch + deletion frequency) SVM accuracy falls in the 88–91% range, consistent with the ~90% in vitro accuracy reported in the paper
- Current-intensity-only SVM accuracy falls in the weaker 43–65% range, confirming current intensity alone is not informative
- Training labels match the oligo design: sites designed with m6A are labeled modified and RRACH sites with A are labeled unmodified
- Feature tables contain per-site values at position 0 and neighboring positions of the target 5-mer

## Limitations

- Feature contributions vary across k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives
- The algorithm does not predict modifications in individual RNA molecules and cannot distinguish between modification types (e.g. m1A vs m6A)
- Only ~32% of known modification sites are recovered overall by the method
- EpiNano-SVM predictions are base-caller version dependent — v1.1 models are only accurate with Albacore 2.1.7 base-called data, and v1.2 models with Guppy 3.1.5
- Direct RNA base-calling produces many 'errors' even on unmodified sequences, so a matched control dataset is highly recommended despite being unnecessary for supervised synthetic training

## Evidence

- [abstract] Synthetic training data with known labels: "our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy"
- [abstract] Base-called features as modification signal: "using direct RNA sequencing, N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
- [readme] What EpiNano extracts from synthetic reads: [evidence span withheld — anchor status: external_doc]
- [readme] SVM models trained on known-site oligos: [evidence span withheld — anchor status: external_doc]
- [discussion] k-mer specificity caveat: "the contribution of each feature varies across different k-mers"
- [discussion] Site-level (not per-read) predictions: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
