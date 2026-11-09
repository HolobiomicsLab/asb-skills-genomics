---
name: synthetic-rna-oligo-read-labeling
description: Use when a single-nucleotide m6A classifier needs read-level ground-truth
  labels from synthetic RNA oligos whose modification status per nucleotide is precisely
  controlled, especially for train-on-RL/test-on-SL generalization tests across unseen
  DRACH contexts.
when_to_use_negative:
- Input is natural mRNA without per-nucleotide modification ground truth (approximate
  labels only, e.g. miCLIP2-style sites) — read-level y=0/y=1 labels cannot be justified
- The goal is site-level stoichiometry on biological samples (use mAFiA site-level
  pileup with min_coverage 50 and P(m6A) threshold 0.5 instead of training labels)
- The target modification is outside m6A in DRACH motifs (e.g. pseudouridine in GUUCN-like
  motifs — use psi-co-mAFiA motifs instead)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: RODAN
  role: Fully convolutional backbone used to basecall RL/SL fast5 reads and emit hidden
    features for the 768-dim vector extraction
  repo: https://github.com/biodlab/RODAN
- name: mAFiA
  role: Pipeline that extracts the 768-dim per-nucleotide feature vector from RODAN's
    convlayers.conv21 and fits the per-motif logistic-regression classifiers
  repo: https://github.com/dieterich-lab/mAFiA
- name: minimap2.22
  role: Aligns basecalled RL/SL reads to reference sequences containing all possible
    random oligo combinations
- name: sklearn
  role: Logistic regression training with the default L-BFGS solver on the 0.75-0.25
    split resampled by 4-fold cross-validation
- name: nanopolish 0.13.3
  role: Resquiggling of reads during feature preparation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:53a00bf903fd37e2014615ecd64665d54c2642db16594a74d5a54c1df9cce0a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodlab/RODAN
  tools:
  - RODAN
  - mAFiA
  - minimap2.22
  - nanopolish 0.13.3
  - sklearn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# Synthetic RNA oligo read-level labeling

## Summary

Generates read-level ground-truth m6A labels by basecalling direct RNA-Seq reads from synthetic UNM/MOD oligo pairs (random-ligation and splint-ligation) with a RODAN backbone, extracting 768-dim features per target-motif nucleotide, and assigning y=0/y=1 labels for classifier training and cross-context validation.

## When to use

Use when a single-nucleotide m6A classifier needs read-level ground-truth labels from synthetic RNA oligos whose modification status per nucleotide is precisely controlled, especially for train-on-RL/test-on-SL generalization tests across unseen DRACH contexts.

## When NOT to use

- Input is natural mRNA without per-nucleotide modification ground truth (approximate labels only, e.g. miCLIP2-style sites) — read-level y=0/y=1 labels cannot be justified
- The goal is site-level stoichiometry on biological samples (use mAFiA site-level pileup with min_coverage 50 and P(m6A) threshold 0.5 instead of training labels)
- The target modification is outside m6A in DRACH motifs (e.g. pseudouridine in GUUCN-like motifs — use psi-co-mAFiA motifs instead)

## Inputs

- Raw ONT fast5 reads from random-ligation (RL) synthetic oligo samples
- Raw ONT fast5 reads from splint-ligation (SL) synthetic oligo samples
- Reference sequences containing all possible random combinations of constituent oligos
- UNM/MOD pairing of the 15 synthetic oligo sequences (m6A inserted at sequence center of MOD variants)

## Outputs

- Read-level feature table of 768-dimensional RODAN convlayers.conv21 vectors per target-motif A nucleotide, labeled y=0 (UNM) / y=1 (MOD)
- Pickled sklearn logistic-regression classifiers
- Per-DRACH-motif precision-recall average-precision (AUC) metrics on the SL held-out set

## How to apply

Basecall RL and SL synthetic-oligo fast5 reads with the RODAN-based backbone, then align basecalls with minimap2.22 and filter with samtools at mapq 50 (choosing the longest continuous mapping if alignment is discontinuous). Extract the 768-dimensional feature vector from RODAN's last convolution layer ('convlayers.conv21') for each read position predicted as an A within one of the six target DRACH motifs. Label features from unmodified (UNM) oligos as y=0 and modified (MOD) oligos as y=1, subsampling the majority class to balance the training set. Train a logistic regression (sklearn default L-BFGS solver) on 75% of RL features with a 0.75-0.25 split resampled by 4-fold cross-validation, and validate on held-out SL features from sequence contexts unseen during training. Pickle the classifiers and record per-motif precision-recall average-precision AUCs as the success metric.

## Related tools

- **RODAN** (Fully convolutional backbone used to basecall RL/SL fast5 reads and emit hidden features for the 768-dim vector extraction) — https://github.com/biodlab/RODAN
- **mAFiA** (Pipeline that extracts the 768-dim per-nucleotide feature vector from RODAN's convlayers.conv21 and fits the per-motif logistic-regression classifiers) — https://github.com/dieterich-lab/mAFiA
- **minimap2.22** (Aligns basecalled RL/SL reads to reference sequences containing all possible random oligo combinations)
- **sklearn** (Logistic regression training with the default L-BFGS solver on the 0.75-0.25 split resampled by 4-fold cross-validation)
- **nanopolish 0.13.3** (Resquiggling of reads during feature preparation)

## Examples

```
minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}
```

## Evaluation signals

- Average precision-recall AUC computed per motif over all six DRACH motifs on the SL held-out set should show robust performance on hitherto unseen sequence contexts
- Class balance: UNM (y=0) vs MOD (y=1) feature counts must be equalized by majority-class subsampling before fitting
- Only read positions whose predicted base is A within one of the six target DRACH motifs should enter the feature table
- Read filtering invariants: no reads with mapq < 50; for discontinuous alignments only the longest continuous mapping section is retained
- Model-selection criterion reproduced: 4-fold cross-validated 0.75-0.25 train-validation split on RL features, matching the paper's setup

## Limitations

- The six most common DRACH motifs covered account for only ~80% of all consensus m6A sites in human mRNA; rare DRACH contexts are not labeled
- Labels come from synthetic oligos (15 sequences, m6A at the sequence center) — modified nucleotides are always at the motif center, so positional generalization is untested by construction
- No read-level database of naturalistic RNA with modification labels exists, so labels cannot be extended directly to native mRNA reads
- Training requires majority-class subsampling because UNM and MOD feature counts are imbalanced; results depend on the sampled split

## Evidence

- [results] UNM features labeled y=0, MOD features labeled y=1; majority class subsampled to balance: [evidence span withheld — anchor status: not_found]
- [results] 768-dim feature extraction from RODAN's last convolution layer: [evidence span withheld — anchor status: not_found]
- [results] 4-fold cross-validation with 0.75-0.25 split: "The 0.75-0.25 train-validation split is resampled by 4-fold cross-validation"
- [results] sklearn logistic regression with default L-BFGS solver: "Optimization is performed with the python module sklearn, using the default L-BFGS solver"
- [results] Read-level mapq filter: "Reads with mapq below 50 are filtered out."
- [readme] RODAN backbone used for basecalling and feature extraction: "backbone.torch: [RODAN](https://github.com/biodlab/RODAN)-based neural network for basecalling and feature extraction"
- [readme] Alignment and q50 filter command from the mAFiA walkthrough: "minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall}"
