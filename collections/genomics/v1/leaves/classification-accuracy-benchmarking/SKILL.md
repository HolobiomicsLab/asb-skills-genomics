---
name: classification-accuracy-benchmarking
description: Use when benchmarking per-feature vs combined-feature classifiers on
  base-called features (mismatch, deletion, base quality, current intensity) extracted
  from m6A-modified vs unmodified RRACH 5-mers, and when validating accuracy against
  published ranges.
when_to_use_negative:
- The goal is single-read modification calling — the article states the algorithm
  'does not predict RNA modifications in individual RNA molecules, but rather employs
  information from all the reads mapping to a specific site'
- The goal is distinguishing modification types (e.g., m1A vs m6A) — the article states
  the algorithm 'does not distinguish between different types of RNA modifications'
- The input is already a feature table and only a single pre-trained model prediction
  is needed (e.g., EpiNano-SVM with Guppy 3.1.5 pre-trained m6A models) rather than
  a feature-by-feature accuracy benchmark
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3168
tools:
- name: EpiNano
  role: Extracts per-base and per-kmer base-calling 'error' features (mismatch, deletion,
    insertion, base quality, current intensity) from ONT direct RNA reads and provides
    the SVM framework used for m6A classification
  repo: https://github.com/novoalab/EpiNano
- name: scikit-learn
  role: Trains and evaluates the individual and combined-feature binary SVM classifiers
    on per-site feature tables
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the base-called reads whose systematic
    errors and quality decreases encode m6A signal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:4bd65e054faf20da2039d4d282c3c134075370f223f09c6ca519929e962e96a6
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# classification-accuracy-benchmarking

## Summary

Benchmarks binary classifiers trained on base-called features from ONT direct RNA sequencing — comparing per-feature SVMs (base quality, mismatch, deletion frequency) against a combined-feature SVM to quantify which base-calling 'error' features discriminate m6A-modified from unmodified RRACH k-mer sites.

## When to use

Use when benchmarking per-feature vs combined-feature classifiers on base-called features (mismatch, deletion, base quality, current intensity) extracted from m6A-modified vs unmodified RRACH 5-mers, and when validating accuracy against published ranges.

## When NOT to use

- The goal is single-read modification calling — the article states the algorithm 'does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site'
- The goal is distinguishing modification types (e.g., m1A vs m6A) — the article states the algorithm 'does not distinguish between different types of RNA modifications'
- The input is already a feature table and only a single pre-trained model prediction is needed (e.g., EpiNano-SVM with Guppy 3.1.5 pre-trained m6A models) rather than a feature-by-feature accuracy benchmark

## Inputs

- Base-called per-site feature tables from EpiNano (mismatch frequency, deletion frequency, insertion frequency, base quality, sumErr, current intensity) at position 0 and neighbor positions for RRACH 5-mers
- Per-site binary labels (m6A-modified vs unmodified) from synthetic RNA spike-in sequences with known modification sites
- FASTQ reads from ONT direct RNA sequencing of modified and unmodified synthetic transcripts

## Outputs

- svm_accuracy_results.csv with per-feature, combined-feature, and current-intensity classification accuracies
- Trained per-feature and combined-feature scikit-learn SVM models

## How to apply

Extract per-site features (mismatch frequency, deletion frequency, base quality, and current intensity) at position 0 and neighboring positions for RRACH 5-mers from m6A-modified and unmodified synthetic direct RNA reads processed through the EpiNano pipeline on Oxford Nanopore data. Train individual binary SVM classifiers in scikit-learn — one per feature — on the modified vs unmodified per-site feature tables, then train a combined-feature SVM using position-0 and neighbor features together. Evaluate each SVM on held-out per-site labels and record accuracies per feature and combined. The rationale is that m6A manifests as systematic base-calling errors and decreased base-calling qualities, so feature-level classification quantifies which of these signals is informative; combined features outperform individual ones (~90% in vitro accuracy reported), while current intensity alone is weakly discriminative.

## Related tools

- **EpiNano** (Extracts per-base and per-kmer base-calling 'error' features (mismatch, deletion, insertion, base quality, current intensity) from ONT direct RNA reads and provides the SVM framework used for m6A classification) — https://github.com/novoalab/EpiNano
- **scikit-learn** (Trains and evaluates the individual and combined-feature binary SVM classifiers on per-site feature tables)
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the base-called reads whose systematic errors and quality decreases encode m6A signal)

## Examples

```
from sklearn.svm import SVC; clf = SVC().fit(X_train, y_train); acc = clf.score(X_test, y_test)
```

## Evaluation signals

- Per-feature SVM accuracies fall in the 70–86% range (base quality, mismatch, deletion frequency individually)
- Combined-feature SVM accuracy (position-0 plus neighbor features) falls in the 88–91% range, consistent with the ~90% in vitro accuracy reported in the paper
- Current-intensity-only accuracy falls in the 43–65% range, confirming it is weakly discriminative on its own
- Combined-feature accuracy exceeds every individual-feature accuracy, replicating the paper's finding that feature combination improves classification
- svm_accuracy_results.csv contains one accuracy row per feature plus combined-feature and current-intensity entries

## Limitations

- The algorithm aggregates information from all reads mapping to a site; per-read RNA modification detection was left to future work
- The algorithm cannot distinguish between different types of RNA modifications (e.g., m1A vs m6A)
- Feature contributions vary across k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives
- Only ~32% of known modification sites are recovered overall despite high classification accuracy
- Pre-trained SVM models are base-caller dependent — EpiNano 1.2 models require Guppy 3.1.5 base-called data, and EpiNano 1.0/1.1 models were accurate only with Albacore 2.1.7

## Evidence

- [abstract] combined-feature accuracy 88-91% vs per-feature baseline: "our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy"
- [discussion] m6A detected as systematic errors and quality decreases: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [discussion] k-mer-specific feature contributions require tailored models: "the contribution of each feature varies across different k-mers"
- [readme] SVM mode supports building custom models: [evidence span withheld — anchor status: not_found]
- [readme] current intensity is extracted but was not used in final models: [evidence span withheld — anchor status: external_doc]
- [readme] pre-trained models trained on modified/unmodified sequences with known sites: [evidence span withheld — anchor status: external_doc]
