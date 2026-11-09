---
name: methylation-motif-classification
description: Use when Nanopore reads from bacteria yield candidate methylation motifs
  whose types (4mC/5mC/6mA) and methylated-base positions must be classified with
  a trained multi-column CNN, especially when nanodisco's mean-feature-only typing
  is insufficient.
when_to_use_negative:
- Input is already modBAM/bedMethyl modified-base calls from a contemporary ONT basecaller
  — use modkit pileup for summarizing those instead of de novo motif typing
- The goal is metagenomic binning of microbiome contigs from methylation profiles
  — that is nanodisco's methylation-binning functionality, not Nanoident motif typing
- No WGA/NAT paired samples are available to derive the five signal-derived features,
  since preprocessing requires subtracting WGA from NAT reads
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
tools:
- name: Nanoident
  role: Multi-column CNN model that performs typing (identification of methylation
    types) and fine mapping (identification of methylated base positions); trained
    and tested via train_loocv_model.py, train_loocv_test.py and train_test.py
  repo: https://github.com/cz-csu/Nanoident
- name: postprocess.py
  role: Position-correction post-processing module applied to raw predictions to achieve
    the best results with module (+6.34% accuracy on the 12-motif independent set)
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: Provides the Singularity (v3.2.1 and above) preprocessing environment for
    extracting features from nanopore signal, and serves as the baseline method for
    accuracy comparison (66.58% on the independent test set)
  repo: https://github.com/fanglab/nanodisco
- name: bam-readcount
  role: Used to get the basecall feature during preprocessing
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:67078c673aba3a337d8ba62426a5f32996f654635b12b6eb8e8c720ab3d35a18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - bam-readcount
  - conda
  - nanodisco
  - postprocess.py
  - train_test.py
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# methylation-motif-classification

## Summary

Nanoident's multi-column convolutional neural network that performs typing (identifying methylation types 4mC/5mC/6mA) and fine mapping (identifying methylated base positions) of bacterial methylation motifs from Nanopore sequencing, improving accuracy ~10% over nanodisco under LOOCV and ~13% on an independent dataset.

## When to use

Use when Nanopore reads from bacteria yield candidate methylation motifs whose types (4mC/5mC/6mA) and methylated-base positions must be classified with a trained multi-column CNN, especially when nanodisco's mean-feature-only typing is insufficient.

## When NOT to use

- Input is already modBAM/bedMethyl modified-base calls from a contemporary ONT basecaller — use modkit pileup for summarizing those instead of de novo motif typing
- The goal is metagenomic binning of microbiome contigs from methylation profiles — that is nanodisco's methylation-binning functionality, not Nanoident motif typing
- No WGA/NAT paired samples are available to derive the five signal-derived features, since preprocessing requires subtracting WGA from NAT reads

## Inputs

- pkl-formatted feature matrices converted from R with data_from_R_to_python.ipynb, containing five Nanoident features (mean, std, time, proportion of reference bases, average quality score) plus the basecalling feature
- Pretrained multi-column CNN checkpoint best_model/Nanoident.pt
- Nanopore signal preprocessed from WGA and NAT sample pairs via the nanodisco/Singularity pipeline (nanodisco and bam-readcount installed first)
- Reference sequences (e.g. NCBI CP041693, CP041696, NC_008261.1) and REBASE methylation motif annotations
- Independent test pkl for the 12 motifs (6 4mC, 1 5mC, 5 6mA) derived from SRA BioProject PRJNA559199

## Outputs

- Methylation type labels (4mC/5mC/6mA) per motif and methylated-base-position predictions (raw and post-processed)
- Accuracy results tables: test_accuracy_results.csv (73.58% no post-processing, 79.92% with module, nanodisco baseline 66.58%) and loocv_accuracy_results.csv (LOOCV 83.68%, Bi-LSTM 71.51%, Transformer 67.61%)

## How to apply

Preprocess original nanopore signal with the nanodisco-based pipeline, but first run update.sh so the workflow extracts Nanoident's four additional features (std, time, proportion of reference bases, average quality score) beyond nanodisco's mean feature, plus a base feature to facilitate positional correction. Build the R training data with Rscript ./code/Nanoident_traindata.R (motifs annotated from REBASE, references from NCBI accessions such as CP041693), then convert R output to pkl format with data_from_R_to_python.ipynb. Train/test the multi-column CNN under leave-one-out cross-validation on the 7-bacteria/46-motif dataset using train_loocv_model.py and train_loocv_test.py (LOOCV accuracy ~83.68%, ~10% above nanodisco), and evaluate the independent 12-motif test set (N. otitidiscaviarum NEB252 and T. phaeum DSM 12270, from SRA BioProject PRJNA559199) with the test-results section of ./model/train_test.py against the pretrained best_model/Nanoident.pt. Finally run postprocess.py to apply the position-correction post-processing module, which raises independent-set accuracy from 73.58% to 79.92% (+6.34%).

## Related tools

- **Nanoident** (Multi-column CNN model that performs typing (identification of methylation types) and fine mapping (identification of methylated base positions); trained and tested via train_loocv_model.py, train_loocv_test.py and train_test.py) — https://github.com/cz-csu/Nanoident
- **postprocess.py** (Position-correction post-processing module applied to raw predictions to achieve the best results with module (+6.34% accuracy on the 12-motif independent set)) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Provides the Singularity (v3.2.1 and above) preprocessing environment for extracting features from nanopore signal, and serves as the baseline method for accuracy comparison (66.58% on the independent test set)) — https://github.com/fanglab/nanodisco
- **bam-readcount** (Used to get the basecall feature during preprocessing)

## Examples

```
python ./model/train_test.py; python postprocess.py
```

## Evaluation signals

- LOOCV accuracy on the 7-bacteria/46-motif dataset should match ~83.68%, exceeding nanodisco and baselines (Bi-LSTM 71.51%, Transformer 67.61%)
- Independent-set accuracy on the 12-motif test set should be 73.58% without post-processing and 79.92% with the position-correction module (+6.34%)
- nanodisco baseline on the same test set should be 66.58%, so Nanoident's gain is ~7.2% (~13% relative improvement)
- Motif composition sanity check: training set has 7 4mC, 11 5mC, 28 6mA motifs; test set has 6 4mC, 1 5mC, 5 6mA motifs
- All expected values should appear in test_accuracy_results.csv and loocv_accuracy_results.csv within a small tolerance

## Limitations

- Original nanodisco could only handle mean features, so update.sh is required before preprocessing; Nanoident handles the four additional features
- Preprocessing must be done in chunks then merged because of memory constraints
- Only the three methylation types 6mA, 5mC and 4mC are supported, matching nanodisco's scope
- The nanodisco container must be built with Singularity v3.2.1 and above, and nanodisco plus bam-readcount must be installed first

## Evidence

- [abstract] We propose a deep-learning method, called Nanoident, for de novo detection of DNA methylation types and methylated base positions in bacteria using Nanopore sequencing.: "We propose a deep-learning method, called Nanoident, for de novo detection of DNA methylation types and methylated base positions in bacteria using Nanopore sequencing."
- [supplementary] you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module: "you can just use the model we provide in best_model/Nanoident.pt, you can run postprocess.py achieve the best results with module"
- [supplementary] There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs.: "There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs."
- [readme] The original nanodisco could only handle mean features, but Nanoident can handle four additional features (std, time,the proportion of reference bases, average quality score).: "The original nanodisco could only handle mean features, but Nanoident can handle four additional features (std, time,the proportion of reference bases, average quality score)."
- [abstract] Nanoident achieves ∼10% improvement in accuracy than the previous method.: "Nanoident achieves ∼10% improvement in accuracy than the previous method."
