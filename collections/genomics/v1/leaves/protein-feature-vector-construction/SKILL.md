---
name: protein-feature-vector-construction
description: Use when starting from raw FASTA protein sequences and you need a fixed-length
  dipeptide-composition feature matrix (400 dimensions per protein) to feed downstream
  feature selection or SVM classification.
when_to_use_negative:
- Input is already a computed feature matrix or workspace .mat file (e.g. feature_NMBAC_594.mat)
  — re-encoding would be redundant
- Sequences are shorter than 50 residues or contain 'X' ambiguous residues, which
  the iDBP-DEP pipeline excludes
- The goal is PSSM-based evolutionary encoding (PSSM-DBT/PSSM-DWT), which requires
  PSI-BLAST profiles rather than raw sequence dipeptide statistics
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: iDBP-DEP (DDE.py)
  role: Python implementation of the DDE dipeptide descriptor, run from Code/feature_Extract/DDE
    on the deposited FASTA datasets
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:b7659a48d30fd5161bff736f0b85f0f7b60897ca88b3818ee125e9a33a0ac4b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - iDBP-DEP (DDE.py)
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protein-feature-vector-construction

## Summary

Constructs per-protein numeric feature vectors for DNA-binding protein prediction by encoding dipeptide composition as the Dipeptide Deviation from Expected Mean (DDE) descriptor, the first encoding step of the iDBP-DEP multi-view feature pipeline before PSSM-DBT, PSSM-DWT and NMBAC encoding.

## When to use

Use when starting from raw FASTA protein sequences and you need a fixed-length dipeptide-composition feature matrix (400 dimensions per protein) to feed downstream feature selection or SVM classification.

## When NOT to use

- Input is already a computed feature matrix or workspace .mat file (e.g. feature_NMBAC_594.mat) — re-encoding would be redundant
- Sequences are shorter than 50 residues or contain 'X' ambiguous residues, which the iDBP-DEP pipeline excludes
- The goal is PSSM-based evolutionary encoding (PSSM-DBT/PSSM-DWT), which requires PSI-BLAST profiles rather than raw sequence dipeptide statistics

## Inputs

- FASTA protein sequences from benchmark DNA-binding datasets (PDB1075, PDB594, PDB186)
- Amino-acid codon count table (61 sense codons, stop codons excluded) for computing theoretical mean/variance

## Outputs

- Per-protein 400-dimension DDE feature vectors
- Per-dataset DDE feature matrices saved as MATLAB workspace .mat files (DDE.mat, DDE594.mat, DDE186.mat / DDE_workspace.mat, DDE594_workspace.mat)

## How to apply

Load FASTA sequences from the deposited PDB1075, PDB594 and PDB186 datasets (in the repo's ~/Data feature-data directories), excluding sequences shorter than 50 residues or containing 'X'. For each sequence compute dipeptide composition Dc(r,s) = N_rs/(N-1) over all 20x20 = 400 amino-acid dipeptides, then compute the theoretical mean Tm(r,s) from amino-acid codon counts (61 sense codons, three stop codons excluded) and the theoretical variance Tv(r,s) = Tm(r,s)*(1-Tm(r,s))/(N-1). Combine them as DDE(r,s) = (Dc(r,s) - Tm(r,s)) / sqrt(Tv(r,s)) to yield one 400-dimension vector per protein. Assemble the vectors into per-dataset feature matrices and save them as MATLAB workspace files (e.g. DDE_workspace.mat, DDE594_workspace.mat) using the repo's Code/feature_Extract/DDE/DDE.py, so they can later be concatenated with PSSM-DBT, PSSM-DWT and NMBAC blocks into the 3240-dimension iDBP-DEP feature vector.

## Related tools

- **iDBP-DEP (DDE.py)** (Python implementation of the DDE dipeptide descriptor, run from Code/feature_Extract/DDE on the deposited FASTA datasets) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
python ~/Code/feature_Extract/DDE/DDE.py --input ~/Data/PDB1075/PDB1075_seqs.fasta --output ~/PDB1075_workspace/DDE_workspace.mat
```

## Evaluation signals

- Every protein in a dataset yields exactly a 400-dimension DDE feature vector (20x20 dipeptides)
- Computed DDE matrices match the deposited workspace files DDE_workspace.mat (PDB1075) and DDE594_workspace.mat (PDB594)
- Theoretical mean/variance use only the 61 sense codons (three stop codons excluded)
- DDE block concatenates with PSSM-DBT, PSSM-DWT (1040-dim) and NMBAC (200-dim) into the expected 3240-dimension iDBP-DEP feature vector

## Limitations

- Proteins shorter than 50 residues or containing 'X' are excluded from the deposited datasets
- DDE is only the first of four encoding views; alone it is insufficient for the full iDBP-DEP representation (3240-dim combined vector)
- No accession IDs or provenance are given for the benchmark datasets PDB1075, PDB594, PDB186

## Evidence

- [methods] we initiatively applied DDE descriptor to represent the dipeptide composition: [evidence span withheld — anchor status: not_found]
