---
name: baseline-method-accuracy-benchmarking
description: Use when a new methylation typing or fine-mapping model needs a fair
  comparison against a prior method on the same held-out motifs, e.g. reproducing
  Nanoident's 83.68% vs nanodisco's 73.93% LOOCV accuracy.
when_to_use_negative:
- Evaluating on the independent 12-motif test dataset of two separate bacteria — that
  requires the pretrained model / test-data workflow, not LOOCV over the 46 training
  motifs
- Input data has not been preprocessed from the original nanopore signal (no WGA/NAT
  differenced, chunked, merged features yet) — run the nanodisco/Nanoident preprocessing
  pipeline first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0654
tools:
- name: Nanoident
  role: Method under test; multi-column CNN whose per-fold models are trained and
    evaluated in the LOOCV benchmark
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: Baseline method retrained under the same LOOCV protocol for accuracy comparison
  repo: https://github.com/fanglab/nanodisco
- name: train_loocv_model.py
  role: Trains one Nanoident CNN per LOOCV fold on the 45 non-held-out motifs
  repo: https://github.com/cz-csu/Nanoident
- name: train_loocv_test.py
  role: Evaluates each fold model on its held-out motif to produce fold-level typing
    accuracy
  repo: https://github.com/cz-csu/Nanoident
- name: data_from_R_to_python.ipynb
  role: Converts R-preprocessed motif data to pkl format usable in the Python training
    environment
  repo: https://github.com/cz-csu/Nanoident
- name: Singularity
  role: Virtualization layer (v3.2.1+) for building the nanodisco baseline environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:3b82cf43a389cfb22e6ebda5ca2019fb5ccd440cb7453bc002fbae4c319f140a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - R
  - Rscript
  - Singularity
  - conda
  - data_from_R_to_python.ipynb
  - nanodisco
  - train_loocv_model.py
  - train_loocv_test.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# baseline-method-accuracy-benchmarking

## Summary

Benchmarks a new deep-learning method (Nanoident) against an established baseline (nanodisco) under an identical Leave-One-Out Cross-Validation (LOOCV) protocol over 46 methylation motifs from seven bacterial species, quantifying per-motif and average typing accuracy.

## When to use

Use when a new methylation typing or fine-mapping model needs a fair comparison against a prior method on the same held-out motifs, e.g. reproducing Nanoident's 83.68% vs nanodisco's 73.93% LOOCV accuracy.

## When NOT to use

- Evaluating on the independent 12-motif test dataset of two separate bacteria — that requires the pretrained model / test-data workflow, not LOOCV over the 46 training motifs
- Input data has not been preprocessed from the original nanopore signal (no WGA/NAT differenced, chunked, merged features yet) — run the nanodisco/Nanoident preprocessing pipeline first

## Inputs

- pkl-format training data for 7 bacterial species covering 46 methylation motifs (7 4mC, 11 5mC, 28 6mA), converted from R with data_from_R_to_python.ipynb
- Nanopore signal-derived features (mean, std, time, proportion of reference bases, average quality score, base feature) from WGA and NAT samples
- REBASE methylation motif annotations for motif labels

## Outputs

- Per-fold held-out motif accuracy from each Nanoident LOOCV model
- Per-motif accuracy table for Nanoident and retrained nanodisco
- Average LOOCV accuracy comparison (expected: Nanoident 83.68% vs nanodisco 73.93%)

## How to apply

Convert the R preprocessed motif data to pkl via data_from_R_to_python.ipynb, covering the 7 bacteria and their 46 methylation motifs (7 4mC, 11 5mC, 28 6mA). For each LOOCV fold, hold out one of the 46 motifs, train the Nanoident multi-column CNN on the remaining 45 with train_loocv_model.py, and score the held-out motif with train_loocv_test.py to get fold-level typing accuracy. Retrain the nanodisco baseline under the same LOOCV protocol (Singularity >= v3.2.1 container, bam-readcount feature extraction) so both methods see identical splits and features. Aggregate fold accuracies into per-motif and average tables for both methods and compare the deltas. Validation target: Nanoident average LOOCV accuracy should reproduce the reported 83.68% and exceed the retrained nanodisco baseline (73.93%), matching the reported ~10% improvement.

## Related tools

- **Nanoident** (Method under test; multi-column CNN whose per-fold models are trained and evaluated in the LOOCV benchmark) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Baseline method retrained under the same LOOCV protocol for accuracy comparison) — https://github.com/fanglab/nanodisco
- **train_loocv_model.py** (Trains one Nanoident CNN per LOOCV fold on the 45 non-held-out motifs) — https://github.com/cz-csu/Nanoident
- **train_loocv_test.py** (Evaluates each fold model on its held-out motif to produce fold-level typing accuracy) — https://github.com/cz-csu/Nanoident
- **data_from_R_to_python.ipynb** (Converts R-preprocessed motif data to pkl format usable in the Python training environment) — https://github.com/cz-csu/Nanoident
- **Singularity** (Virtualization layer (v3.2.1+) for building the nanodisco baseline environment)

## Examples

```
python ./model/train_loocv_model.py && python ./model/train_loocv_test.py
```

## Evaluation signals

- Nanoident average LOOCV accuracy over the 46 motifs reproduces the reported 83.68%
- Retrained nanodisco baseline reaches approximately 73.93% average LOOCV accuracy
- Nanoident's average accuracy exceeds nanodisco's by roughly 10 percentage points
- All 46 motifs (7 4mC, 11 5mC, 28 6mA) appear exactly once as the held-out fold, with no fold leakage
- Per-motif accuracy table covers all 46 motifs for both methods with comparable fold splits

## Limitations

- The original nanodisco could only handle mean features, so update.sh is required to add std, time, proportion-of-reference-bases, and average quality score features before a fair baseline comparison
- Results are specific to the 7-bacterium / 46-motif training set; the independent 12-motif test set (6 4mC, 1 5mC, 5 6mA) shows a different improvement (~13%)
- Preprocessing must be chunked due to memory constraints before merging WGA/NAT subtracted files

## Evidence

- [supplementary] LOOCV accuracy comparison target: [evidence span withheld — anchor status: not_found]
- [supplementary] Motif composition of training set: "There are 46 different motifs in these seven bacteria, including 7 4mC motifs, 11 5mC motifs, and 28 6mA motifs."
- [methods] LOOCV scripts in repository: "python ./model/train_loocv_model.py"
- [methods] Baseline feature update requirement: "The original nanodisco could only handle mean features, but Nanoident can handle four additional features"
- [abstract] Headline improvement claim: "The Leave-One-Out Cross-Validation (LOOCV) on the dataset containing 7 bacteria samples with 46 methylation motifs shows that Nanoident achieves ∼10% improvement in accuracy than the previous method."
- [readme] Python environment setup: "Use Nanoident/env/freeze.yml to generate the environment directly under conda."
