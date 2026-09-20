---
name: evolutionary-profile-encoding
description: Use when input is a set of protein sequences with PSI-BLAST PSSM profiles
  (N x 20 matrices) and you need fixed-length evolutionary feature vectors (e.g. 1600-D
  PSSM-DBT or 1040-D PSSM-DWT) to feed an SVM classifier.
when_to_use_negative:
- Input is already an encoded feature table (e.g. the repo's DDE.mat, PSSM_DBT.mat,
  PSSM_DWT.mat files) rather than raw PSSM text profiles
- Sequences lack PSI-BLAST profiles and computing PSSMs against nr is not feasible,
  since both DBT and DWT operate on the N x 20 PSSM matrix, not the raw sequence
- You need per-residue evolutionary signals rather than fixed-length whole-protein
  descriptors, since DBT and DWT collapse each sequence into one vector
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: PSI-BLAST
  role: Generates the PSSM evolutionary profiles (searched against nr) that are the
    raw input for both DBT and DWT encoding
- name: getDBT.m / ExDBT.m / DBT186.m
  role: MATLAB scripts in Code/feature_Extract/PSSM-DBT that compute the Distance-Bigram
    Transformation features from parsed PSSM files
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: getDWT.m / EstraggoConDWT.m
  role: MATLAB scripts in Code/feature_Extract/PSSM-DWT that perform the 4-scale discrete
    wavelet decomposition and DCT extraction on PSSM signals
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:30c51f29be97c715d9b5b767ffbd5ceac47c04645f2aa83ae27dfae16842edd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PSI-BLAST
  - getDBT.m / ExDBT.m / DBT186.m
  - getDWT.m / EstraggoConDWT.m
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# evolutionary-profile-encoding

## Summary

Encode PSI-BLAST PSSM evolutionary profiles into fixed-length descriptors — Distance-Bigram Transformation (PSSM-DBT, 1600-D) and Discrete Wavelet Transformation (PSSM-DWT, 1040-D) — as used in iDBP-DEP for DNA-binding protein detection.

## When to use

Use when input is a set of protein sequences with PSI-BLAST PSSM profiles (N x 20 matrices) and you need fixed-length evolutionary feature vectors (e.g. 1600-D PSSM-DBT or 1040-D PSSM-DWT) to feed an SVM classifier.

## When NOT to use

- Input is already an encoded feature table (e.g. the repo's DDE.mat, PSSM_DBT.mat, PSSM_DWT.mat files) rather than raw PSSM text profiles
- Sequences lack PSI-BLAST profiles and computing PSSMs against nr is not feasible, since both DBT and DWT operate on the N x 20 PSSM matrix, not the raw sequence
- You need per-residue evolutionary signals rather than fixed-length whole-protein descriptors, since DBT and DWT collapse each sequence into one vector

## Inputs

- Protein FASTA sequences from benchmark datasets PDB1075, PDB594, or PDB186
- PSI-BLAST PSSM text profiles (N x 20 matrices, 3 iterations, e-value 0.001 against nr)
- PSSM text files loaded via Read_Text_files_PSSM.m

## Outputs

- 1600-dimension PSSM-DBT feature vector per protein (4 distances x 400 amino-acid pairs)
- 1040-dimension PSSM-DWT feature vector per protein (20 signals x 52 features)
- Serialized MATLAB .mat workspaces: PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, PSSM_DBT_594_workspace.mat, PSSM_DWT_594_workspace.mat
- Combined 3240-dimension feature vector when concatenated with DDE and NMBAC (200-D) descriptors

## How to apply

First generate or load PSSM profiles by searching target proteins against the non-redundant (nr) database with PSI-BLAST (3 iterations, e-value 0.001), parsing the N x 20 text matrices with the repo's Read_Text_files_PSSM.m loader. For PSSM-DBT, compute the bigram occurrence frequency I(Rx, Ry | t) = (1/(N-t)) * sum_i P[i,x] * P[i+t,y] for all 400 amino-acid pairs at each distance t in {0,1,2,3} — T is set to 3 per 10-fold cross-validation — and concatenate the four 400-D blocks into a 1600-D vector per protein. For PSSM-DWT, treat the PSSM as 20 discrete signals of length N, decompose each 4 times, and at each scale take the maximum, minimum, mean and standard deviation of both approximation and detailed coefficients (32 features) plus the top five DCT coefficients of the approximation coefficients (20 more), yielding 20 x 52 = 1040 dimensions. Serialize the encoded matrices as MATLAB .mat workspace files (e.g. PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, and *_594 variants for PDB594) so they can be combined with DDE and NMBAC into the full 3240-D feature vector.

## Related tools

- **PSI-BLAST** (Generates the PSSM evolutionary profiles (searched against nr) that are the raw input for both DBT and DWT encoding)
- **getDBT.m / ExDBT.m / DBT186.m** (MATLAB scripts in Code/feature_Extract/PSSM-DBT that compute the Distance-Bigram Transformation features from parsed PSSM files) — https://githup.com/Zll-codeside/iDBP-DEP
- **getDWT.m / EstraggoConDWT.m** (MATLAB scripts in Code/feature_Extract/PSSM-DWT that perform the 4-scale discrete wavelet decomposition and DCT extraction on PSSM signals) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
matlab -batch "addpath('Code/feature_Extract/PSSM-DBT'); getDBT"
```

## Evaluation signals

- Each protein yields exactly 1600 PSSM-DBT features (4 distances x 400 bigram pairs) with T=3
- Each protein yields exactly 1040 PSSM-DWT features (20 signals x 52 features: 32 statistics + 5 top DCT coefficients per signal, summed over scales)
- Concatenating DDE + PSSM-DBT + PSSM-DWT + NMBAC produces exactly the 3240-dimension feature vector reported for iDBP-DEP
- Serialized workspace file names and structure match the README layout (e.g. PSSM_DBT_workspace.mat under PDB1075_workspace, PSSM_DBT_594_workspace.mat under PDB594_workspace)
- On PDB1075 with post-feature-selection, PSSM-DBT alone should remain the strongest individual descriptor, reproducing Jackknife Acc of 82.70% and MCC of 0.654

## Limitations

- The DBT dimensionality depends on T=3 (chosen via 10-fold cross-validation); other T values change the feature count, and NMBAC pairing requires nlag=30
- PSSM profiles depend on PSI-BLAST parameters (3 iterations, e-value 0.001) and the nr database state; no PSI-BLAST or LIBSVM version numbers are stated in the article
- DBT/DWT encoding code is MATLAB (.m), while DDE is Python (DDE.py), so the full feature pipeline spans both environments

## Evidence

- [intro] Full 3240-D multi-view vector combines PSSM-DWT (1040-D) and NMBAC (200-D) with other descriptors: [evidence span withheld — anchor status: not_found]
