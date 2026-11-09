---
name: fasta-sequence-quality-filtering
description: Use when preparing raw FASTA protein sequences (e.g. PDB-derived DBP/non-DBP
  benchmark sets) for descriptor extraction and you must remove sequences shorter
  than 50 residues or containing ambiguous 'X' residues.
when_to_use_negative:
- Input is already a feature table or .mat workspace (e.g. DDE594.mat, PSSM_DBT_594.mat)
  — filtering operates on raw FASTA sequences, not on encoded feature matrices
- Sequences are nucleotide or non-protein FASTA records — the 50-residue and 'X'-residue
  criteria are defined for the 20 standard amino acids
- You need sequences shorter than 50 residues preserved, e.g. when studying short
  peptides outside the iDBP-DEP DBP/non-DBP benchmark scope
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: iDBP-DEP (DDE.py / repository scripts)
  role: Provides the ~/Data directories of FASTA sequences and the feature_Extract
    code (e.g. Code/feature_Extract/DDE/DDE.py) that consumes the quality-filtered
    sequences
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:c0a90526a25f40fe6752a467002c195a40f42546090eb7c0654a020d969fdd44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - iDBP-DEP (DDE.py / repository scripts)
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fasta-sequence-quality-filtering

## Summary

Quality filtering of FASTA protein sequences prior to feature encoding in the iDBP-DEP pipeline: deposited PDB1075, PDB594 and PDB186 benchmark sequences are loaded from ~/Data feature-data directories and sequences shorter than 50 residues or containing the ambiguous residue 'X' are excluded before DDE/PSSM feature extraction.

## When to use

Use when preparing raw FASTA protein sequences (e.g. PDB-derived DBP/non-DBP benchmark sets) for descriptor extraction and you must remove sequences shorter than 50 residues or containing ambiguous 'X' residues.

## When NOT to use

- Input is already a feature table or .mat workspace (e.g. DDE594.mat, PSSM_DBT_594.mat) — filtering operates on raw FASTA sequences, not on encoded feature matrices
- Sequences are nucleotide or non-protein FASTA records — the 50-residue and 'X'-residue criteria are defined for the 20 standard amino acids
- You need sequences shorter than 50 residues preserved, e.g. when studying short peptides outside the iDBP-DEP DBP/non-DBP benchmark scope

## Inputs

- FASTA protein sequence files from the deposited benchmark datasets (PDB1075, PDB594, PDB186) stored in the ~/Data feature-data directories of the iDBP-DEP repository

## Outputs

- Filtered set of protein sequences (each >= 50 residues, no 'X' characters) ready for descriptor extraction (DDE, PSSM-DBT, PSSM-DWT, NMBAC)
- Per-dataset sequence lists aligned with the deposited feature matrices (e.g. DDE_workspace.mat, DDE594_workspace.mat)

## How to apply

Load the FASTA protein sequences of the deposited PDB1075, PDB594 and PDB186 datasets from the ~/Data feature-data directories of the iDBP-DEP repository. Parse each FASTA record and apply two exclusion criteria: drop any sequence shorter than 50 residues and drop any sequence whose amino-acid string contains the ambiguous character 'X'. Retain the filtered sequences as the input corpus for downstream per-protein encodings (DDE dipeptide composition, PSSM-DBT, PSSM-DWT, NMBAC), so that every protein passed to feature extraction yields a well-defined descriptor vector (e.g. a 400-dimension DDE vector). Rationale: dipeptide-based descriptors such as DDE(r,s) = (Dc(r,s) - Tm(r,s)) / sqrt(Tv(r,s)) and PSSM-derived encodings assume full-length, unambiguous sequences; short or 'X'-containing entries would produce degenerate or undefined features.

## Related tools

- **iDBP-DEP (DDE.py / repository scripts)** (Provides the ~/Data directories of FASTA sequences and the feature_Extract code (e.g. Code/feature_Extract/DDE/DDE.py) that consumes the quality-filtered sequences) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
python Code/feature_Extract/DDE/DDE.py --input ~/Data/PDB594/*.fasta --min_len 50 --exclude_X --out DDE594_workspace.mat
```

## Evaluation signals

- Every retained sequence has length >= 50 residues and contains no 'X' character
- The number of filtered sequences per dataset matches the deposited benchmark sizes: PDB1075 (525 DBPs + 550 non-DBPs), PDB594 (297 + 297), PDB186 (93 + 93)
- Each filtered protein yields a valid 400-dimension DDE feature vector when passed through the DDE encoding step
- Reconstructed feature matrices align row-for-row with the deposited MATLAB workspace files (DDE_workspace.mat, DDE594_workspace.mat) in the iDBP-DEP repository

## Limitations

- The 50-residue minimum and 'X'-exclusion criteria are specific to the iDBP-DEP pipeline; other DBP predictors may use different cutoffs, so filtered sets are not directly comparable across studies
- No accession IDs or provenance are given for PDB1075, PDB594 and PDB186 in the article's discussion, so re-deriving the exact filtered sequence sets from PDB may be ambiguous
- Filtering discards ambiguous sequences entirely rather than imputing 'X' residues, which may bias against low-quality PDB entries

## Evidence

- [methods] workflow step 1: [evidence span withheld — anchor status: not_found]
- [methods] validation: [evidence span withheld — anchor status: not_found]
