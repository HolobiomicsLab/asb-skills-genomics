---
name: codon-usage-theoretical-mean-computation
description: Use when encoding protein sequences as dipeptide composition and a codon-usage-based
  expected mean/variance is needed to normalize observed dipeptide frequencies, as
  in the DDE descriptor of iDBP-DEP.
when_to_use_negative:
- The input is a PSSM profile or physicochemical-property matrix — use PSSM-DBT/PSSM-DWT
  or NMBAC descriptors instead
- Sequences are shorter than 50 residues or contain ambiguous 'X' residues, which
  are excluded in the iDBP-DEP pipeline
- A plain dipeptide frequency vector (without codon-usage normalization) is the target
  descriptor
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DDE.py (iDBP-DEP)
  role: Python script in ~/Code/feature_Extract/DDE that implements the DDE computation,
    including the codon-usage theoretical mean and variance terms
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:4f64efd532b18e4ebfe4fa835bb1f47bc6b132a7838c6f405b32dac94cb814db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DDE.py (iDBP-DEP)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# codon-usage-theoretical-mean-computation

## Summary

Computes the theoretical mean Tm(r,s) from amino-acid codon counts (61 sense codons, stop codons excluded) used inside the Dipeptide Deviation from Expected Mean (DDE) descriptor, which iDBP-DEP applies to encode dipeptide composition of DNA-binding protein sequences.

## When to use

Use when encoding protein sequences as dipeptide composition and a codon-usage-based expected mean/variance is needed to normalize observed dipeptide frequencies, as in the DDE descriptor of iDBP-DEP.

## When NOT to use

- The input is a PSSM profile or physicochemical-property matrix — use PSSM-DBT/PSSM-DWT or NMBAC descriptors instead
- Sequences are shorter than 50 residues or contain ambiguous 'X' residues, which are excluded in the iDBP-DEP pipeline
- A plain dipeptide frequency vector (without codon-usage normalization) is the target descriptor

## Inputs

- FASTA protein sequences from the PDB1075, PDB594 and PDB186 benchmark datasets (residue length >= 50, no 'X' characters)
- Standard genetic-code codon counts C_r for the 20 amino acids over 61 sense codons

## Outputs

- 400-dimension DDE feature vector per protein
- Per-dataset feature matrices saved as MATLAB workspace files (DDE.mat / DDE594.mat / DDE186.mat, DDE_workspace.mat, DDE594_workspace.mat)

## How to apply

Load FASTA protein sequences (excluding sequences shorter than 50 residues or containing 'X'). Compute the observed dipeptide composition Dc(r,s) = N_rs/(N-1) for all 20x20 = 400 dipeptides. For each dipeptide, derive the theoretical mean Tm(r,s) from the codon counts C_r and C_s of amino acids r and s over the 61 sense codons (the three stop codons excluded), and the theoretical variance Tv(r,s) = Tm(r,s)*(1-Tm(r,s))/(N-1). Combine as DDE(r,s) = (Dc(r,s) - Tm(r,s)) / sqrt(Tv(r,s)) to yield one 400-dimension feature vector per protein, the first encoding step before PSSM encoding with DBT. Assemble per-protein vectors into per-dataset matrices and save as MATLAB workspace files (e.g. DDE_workspace.mat, DDE594_workspace.mat), mirroring the deposited ~/Data feature Data and ~/PDB*_workspace files.

## Related tools

- **DDE.py (iDBP-DEP)** (Python script in ~/Code/feature_Extract/DDE that implements the DDE computation, including the codon-usage theoretical mean and variance terms) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
python Code/feature_Extract/DDE/DDE.py --input Data/PDB1075_seqs_PSSM/../sequences.fasta --output DDE_workspace.mat
```

## Evaluation signals

- Every protein yields exactly a 400-dimension DDE feature vector (20x20 amino-acid dipeptides)
- Tm(r,s) values derive from 61 sense codons — stop codons must be excluded from codon counts
- Sanity check DDE(r,s) = (Dc(r,s) - Tm(r,s)) / sqrt(Tv(r,s)) on a sample sequence against hand computation
- Output workspace matches the deposited DDE.mat / DDE594.mat / DDE186.mat and DDE_workspace.mat / DDE594_workspace.mat files in the iDBP-DEP repository
- Resulting DDE matrix, concatenated with PSSM-DBT, PSSM-DWT and NMBAC, contributes to the 3240-dimension combined feature vector

## Limitations

- DDE is only one of four descriptors; alone it may be insufficient to distinguish DBPs from non-DBPs — iDBP-DEP applies SVM-RFE+CBR feature selection before SVM classification
- Sequences shorter than 50 residues or containing 'X' must be excluded, so the descriptor is not applicable to such sequences without preprocessing
- No accession IDs or provenance are given for the PDB1075, PDB594 and PDB186 benchmark datasets

## Evidence

- [methods] DDE applied to dipeptide composition: [evidence span withheld — anchor status: not_found]
