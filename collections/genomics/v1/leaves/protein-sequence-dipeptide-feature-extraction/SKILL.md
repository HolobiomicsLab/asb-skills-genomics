---
name: protein-sequence-dipeptide-feature-extraction
description: Use when raw protein sequences (e.g. FASTA from PDB1075, PDB594, PDB186)
  must be converted into fixed-length dipeptide-composition feature vectors for machine-learning
  classification of DNA-binding proteins.
when_to_use_negative:
- The input is already a numeric feature matrix or a precomputed DDE workspace (.mat)
  file rather than raw sequences
- The target descriptor is the PSSM-based feature (PSSM-DBT/PSSM-DWT) or the physicochemical
  NMBAC descriptor, which require PSI-BLAST PSSM profiles or amino-acid physicochemical
  property tables instead of bare sequences
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_0080
tools:
- name: iDBP-DEP
  role: Pipeline whose first feature-encoding step expresses dipeptide composition
    using the DDE descriptor; provides the DDE.py extraction script
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:2c81ccf522bcb412865ac988ef52becff297e336c21d5ee22646c7840e6cb69c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - iDBP-DEP
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3510
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# protein-sequence-dipeptide-feature-extraction

## Summary

Computes the Dipeptide Deviation from Expected Mean (DDE) descriptor, a 400-dimension encoding of protein dipeptide composition, as used as the first feature-extraction step of the iDBP-DEP DNA-binding protein predictor.

## When to use

Use when raw protein sequences (e.g. FASTA from PDB1075, PDB594, PDB186) must be converted into fixed-length dipeptide-composition feature vectors for machine-learning classification of DNA-binding proteins.

## When NOT to use

- The input is already a numeric feature matrix or a precomputed DDE workspace (.mat) file rather than raw sequences
- The target descriptor is the PSSM-based feature (PSSM-DBT/PSSM-DWT) or the physicochemical NMBAC descriptor, which require PSI-BLAST PSSM profiles or amino-acid physicochemical property tables instead of bare sequences

## Inputs

- FASTA protein sequences from benchmark datasets PDB1075, PDB594 or PDB186 (sequences shorter than 50 residues or containing 'X' are excluded)

## Outputs

- Per-protein 400-dimension DDE feature vectors
- Per-dataset DDE feature matrices saved as MATLAB workspace files (e.g. DDE.mat, DDE594.mat, DDE_workspace.mat, DDE594_workspace.mat)

## How to apply

Load FASTA protein sequences (the iDBP-DEP pipeline excludes sequences shorter than 50 residues or containing 'X'), then compute the dipeptide composition Dc(r,s) = N_rs/(N-1) for all 20x20 = 400 amino-acid dipeptides. Derive the theoretical mean Tm(r,s) from the codon counts C_r and C_s of amino acids r and s over the 61 sense codons (the three stop codons excluded), and the theoretical variance Tv(r,s) = Tm(r,s)*(1-Tm(r,s))/(N-1). Combine the terms as DDE(r,s) = (Dc(r,s) - Tm(r,s)) / sqrt(Tv(r,s)) so each protein yields one 400-dimension vector. Assemble the per-protein vectors into per-dataset feature matrices and save them as MATLAB workspace files (e.g. DDE_workspace.mat, DDE594_workspace.mat), mirroring the repository's ~/Code/feature_Extract/DDE/DDE.py script. DDE is the first of four iDBP-DEP descriptors, later combined with PSSM-DBT, PSSM-DWT and NMBAC into a 3240-dimension feature vector before SVM-RFE+CBR feature selection.

## Related tools

- **iDBP-DEP** (Pipeline whose first feature-encoding step expresses dipeptide composition using the DDE descriptor; provides the DDE.py extraction script) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
python Code/feature_Extract/DDE/DDE.py  # compute 400-dim DDE vectors from FASTA sequences and save DDE_workspace.mat
```

## Evaluation signals

- Every protein yields exactly a 400-dimension DDE feature vector (20x20 dipeptides)
- Per-dataset DDE matrices match the deposited workspace files (DDE_workspace.mat for PDB1075, DDE594_workspace.mat for PDB594, DDE186.mat for PDB186) in the iDBP-DEP repository
- Dc(r,s) values are non-negative and bounded above by 1/(N-1) since N_rs <= N-1 for a sequence of length N
- Theoretical mean/variance terms use exactly the 61 sense codons, with the three stop codons excluded
- Resulting DDE features concatenate with PSSM-DBT, PSSM-DWT (1040-dim) and NMBAC (200-dim) into the expected 3240-dimension iDBP-DEP vector

## Limitations

- Sequences shorter than 50 residues or containing ambiguous residue 'X' must be excluded from the datasets before encoding
- DDE is only one of four iDBP-DEP views; alone it does not capture evolutionary (PSSM) or physicochemical information needed for the full predictor

## Evidence

- [methods] DDE descriptor applied to represent dipeptide composition in methods: [evidence span withheld — anchor status: not_found]
- [intro] 3240-dimension combined feature vector in the iDBP-DEP pipeline: [evidence span withheld — anchor status: not_found]
