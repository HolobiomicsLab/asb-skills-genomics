---
name: one-hot-encoding-of-genomic-sequences
description: Use when raw genomic splice-site sequences (acceptor/donor, fixed length
  400) must be converted to numeric input before feeding CNNSplice or other CNN-based
  models that require one-hot encoding.
when_to_use_negative:
- Input data is already a numeric feature matrix or previously one-hot encoded (.npy)
  — re-encoding would corrupt shapes
- Sequences have variable or non-400 lengths without prior trimming/padding, since
  the CNNSplice constructed dataset permits a Sequence Length of 400
- The target model accepts raw sequence strings or a different encoding scheme (e.g.
  k-mer features) rather than per-nucleotide one-hot vectors
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3474
tools:
- name: CNNSplice
  role: CNN splice site prediction pipeline whose training and testing scripts require
    genomic sequence input data transformed using one-hot encoding
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: oluwadarelab/cnnsplice:latest
  role: Docker image providing the Python3 environment and dependencies for running
    the one-hot encoding and downstream CNNSplice training/testing
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  - task_001
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:599d279d7a93a50af05df12669835d33ece12cd6cc0ef41ed3f1f5351585f28d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - DeepSplicer
  - Docker
  - GeneSplicer
  - Python
  - Splice2Deep
  - SpliceFinder
  - SpliceRover
  - StratifiedKFold
  - oluwadarelab/cnnsplice:latest
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# one-hot-encoding-of-genomic-sequences

## Summary

Transforms raw genomic nucleotide sequences into fixed-shape binary matrices for CNN input, as required by CNNSplice's splice site prediction models, mapping A, C, G, T (and N) to 4-element binary vectors.

## When to use

Use when raw genomic splice-site sequences (acceptor/donor, fixed length 400) must be converted to numeric input before feeding CNNSplice or other CNN-based models that require one-hot encoding.

## When NOT to use

- Input data is already a numeric feature matrix or previously one-hot encoded (.npy) — re-encoding would corrupt shapes
- Sequences have variable or non-400 lengths without prior trimming/padding, since the CNNSplice constructed dataset permits a Sequence Length of 400
- The target model accepts raw sequence strings or a different encoding scheme (e.g. k-mer features) rather than per-nucleotide one-hot vectors

## Inputs

- Raw genomic nucleotide splice-site sequences (acceptor/donor) of fixed sequence length 400, containing A, C, G, T, and N characters
- CNNSplice dataset files for balanced (5000 true / 5000 false) or imbalanced (7500 true / 2500 false) acceptor and donor datasets

## Outputs

- one_hot_encoded_sequences.npy: collection of one-hot encoded matrices, each of shape (400, 4)
- Numeric CNN input arrays ready for CNNSplice train.py / test.py

## How to apply

Map each nucleotide character to its fixed 4-element binary vector: A to [1 0 0 0], C to [0 1 0 0], G to [0 0 1 0], T to [0 0 0 1], and N to the all-zero vector [0 0 0 0]. Stack the per-position vectors into a Z x 4 input matrix for each sequence, respecting the dataset's fixed sequence length of 400, so every encoded matrix has shape (400, 4). Load the raw sequences using Python3 (CNNSplice is developed in Python3; dependencies come from requirement.txt or the oluwadarelab/cnnsplice Docker image). Save the collection of one-hot encoded matrices to 'one_hot_encoded_sequences.npy' for downstream CNN training/testing. The rationale is that CNNs require numeric input, and CNNSplice's training and testing scripts (train.py, test.py) explicitly expect one-hot encoded genomic sequence input in balanced or imbalanced mode.

## Related tools

- **CNNSplice** (CNN splice site prediction pipeline whose training and testing scripts require genomic sequence input data transformed using one-hot encoding) — https://github.com/OluwadareLab/CNNSplice.git
- **oluwadarelab/cnnsplice:latest** (Docker image providing the Python3 environment and dependencies for running the one-hot encoding and downstream CNNSplice training/testing)

## Examples

```
python train.py -n "output_name" -m "balanced"
```

## Evaluation signals

- Each encoded matrix has shape (400, 4), matching the fixed sequence length of 400
- Each matrix row contains only the five permitted binary patterns: [1 0 0 0], [0 1 0 0], [0 0 1 0], [0 0 0 1], or the all-zero N vector [0 0 0 0]
- No non-ACGTN character was silently dropped during mapping
- The output file 'one_hot_encoded_sequences.npy' loads successfully and its count matches the number of input sequences
- Downstream CNNSplice runs (train.py/test.py) accept the encoded data without shape errors in balanced or imbalanced mode

## Limitations

- The encoding assumes the dataset's fixed sequence length of 400; sequences of other lengths will not conform to the expected (400, 4) matrix shape
- Ambiguous nucleotides other than N are not specified in the mapping, and N encodes to an all-zero vector, so some positional information is lost
- CNNSplice's README notes genomic sequence input data should be transformed using one-hot encoding and that a log directory must exist for output text file storage

## Evidence

- [results] CNNSplice one-hot encodes each nucleotide as a 4-element binary vector: [1 0 0 0] for A, [0 1 0 0] for C, [0 0 1 0] for G, and [0 0 0 1] for T.: [evidence span withheld — anchor status: not_found]
- [methods] Genomic sequence input data should be transfomed using one-hot encoding.: "Genomic sequence input data should be transfomed using one-hot encoding."
- [readme] Our constructed dataset permits a **Sequence Length of 400**: "Our constructed dataset permits a **Sequence Length of 400**"
- [results] Map each nucleotide character to its fixed 4-element binary vector ... and N to the all-zero vector [0 0 0 0].: [evidence span withheld — anchor status: not_found]
