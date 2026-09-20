---
name: fixed-length-sequence-vectorization
description: Use when raw nucleotide sequences (ACGTN) must be transformed into numerical
  input for CNN-based splice site prediction, and the model expects a fixed sequence
  length of 400 per input matrix.
when_to_use_negative:
- Input data is already a numerical feature table or already one-hot encoded — re-encoding
  would be redundant
- Sequences have variable or non-400 lengths that cannot be trimmed/padded to the
  CNNSplice constraint of a sequence length of 400
- The downstream model is not one of the CNN architectures that expects Z x 4 one-hot
  matrices (e.g. MM-based tools like GeneSplicer)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CNNSplice
  role: CNN splice site prediction pipeline whose models consume the one-hot encoded
    (400, 4) sequence matrices produced by this skill
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: CNNSplice web server
  role: Public web interface to CNNSplice models that accept the same preprocessed
    sequence inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:d13514d1ba8331c0e5a6ec062794c46de465fec4009ab1b00447e093890b95b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - CNNSplice web server
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fixed-length-sequence-vectorization

## Summary

Converts fixed-length genomic nucleotide sequences (here, 400 nt splice-site windows) into per-position one-hot binary vectors so they can be fed as numerical input matrices to the CNNSplice CNN models.

## When to use

Use when raw nucleotide sequences (ACGTN) must be transformed into numerical input for CNN-based splice site prediction, and the model expects a fixed sequence length of 400 per input matrix.

## When NOT to use

- Input data is already a numerical feature table or already one-hot encoded — re-encoding would be redundant
- Sequences have variable or non-400 lengths that cannot be trimmed/padded to the CNNSplice constraint of a sequence length of 400
- The downstream model is not one of the CNN architectures that expects Z x 4 one-hot matrices (e.g. MM-based tools like GeneSplicer)

## Inputs

- Raw genomic nucleotide sequence strings (ACGTN) from the CNNSplice splice site datasets (e.g. Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, Caenorhabditis elegans)
- Dataset with fixed sequence length of 400 per splice site window

## Outputs

- Per-sequence one-hot encoded matrix of shape (400, 4)
- Collection of one-hot encoded matrices saved as 'one_hot_encoded_sequences.npy'

## How to apply

Load the raw genomic sequence data in Python3 and map each nucleotide character to a fixed 4-element binary vector: A to [1 0 0 0], C to [0 1 0 0], G to [0 0 1 0], T to [0 0 0 1], and N to the all-zero vector [0 0 0 0]. Stack the per-position vectors into a Z x 4 matrix per sequence, enforcing the CNNSplice dataset's fixed sequence length of 400. Save the stacked one-hot matrices to 'one_hot_encoded_sequences.npy' for downstream CNN training/testing via train.py/test.py. Rationale: the CNNSplice README states genomic sequence input data must be transformed using one-hot encoding, and the constructed dataset permits only a sequence length of 400.

## Related tools

- **CNNSplice** (CNN splice site prediction pipeline whose models consume the one-hot encoded (400, 4) sequence matrices produced by this skill) — https://github.com/OluwadareLab/CNNSplice.git
- **CNNSplice web server** (Public web interface to CNNSplice models that accept the same preprocessed sequence inputs)

## Examples

```
import numpy as np; M = np.array([[{'A':[1,0,0,0],'C':[0,1,0,0],'G':[0,0,1,0],'T':[0,0,0,1],'N':[0,0,0,0]}[c] for c in seq] for seq in seqs]); np.save('one_hot_encoded_sequences.npy', M)
```

## Evaluation signals

- Each encoded matrix has shape (400, 4), matching the dataset's fixed sequence length of 400
- Every row of the encoded matrices is one of the five permitted binary patterns: [1 0 0 0], [0 1 0 0], [0 0 1 0], [0 0 0 1], or [0 0 0 0]
- No non-ACGTN character was silently dropped — every position in the input maps to exactly one row of the output matrix
- Output file 'one_hot_encoded_sequences.npy' loads in Python3 (numpy) and preserves per-sequence ordering for downstream train.py/test.py usage

## Limitations

- Only A, C, G, T, and N are handled; other ambiguity codes (e.g. R, Y) have no defined encoding and may be silently dropped if unvalidated
- The encoding is tied to a fixed sequence length of 400; sequences of other lengths require trimming or padding before encoding
- N positions carry no positional signal (all-zero vector), so unknown bases are effectively zero-padded in the CNN input

## Evidence

- [methods] One-hot encoding mapping: "Genomic sequence input data should be transfomed using one-hot encoding."
- [readme] Sequence length constraint: "Our constructed dataset permits a **Sequence Length of 400**"
- [readme] One-hot encoding requirement in README: "Genomic sequence input data should be transfomed using one-hot encoding."
- [readme] Dataset constraint note: "Dataset sequence length is 400."
