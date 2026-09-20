---
name: protein-sequence-feature-vectorization
description: Use when raw protein sequences (PDB1075/PDB594/PDB186 style FASTA) must
  be turned into fixed-length numeric descriptors before machine-learning classification,
  and PSI-BLAST PSSM profiles plus physicochemical properties are available.
when_to_use_negative:
- Input is already a per-protein feature matrix (e.g. the ~/Data/*/Feature Data .mat
  files) — re-vectorization would duplicate the encoding step
- No PSI-BLAST PSSM profile can be generated (e.g. no nr database access) — PSSM-DBT
  and PSSM-DWT descriptors cannot be computed, only DDE and NMBAC apply
- The downstream task requires residue-level rather than sequence-level fixed-length
  representations
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: PSI-BLAST
  role: Generates the N x 20 PSSM evolutionary profiles against nr (3 iterations,
    e-value 0.001) that are the substrate for the PSSM-DBT and PSSM-DWT descriptors
- name: iFeature
  role: Referenced feature-generation toolkit context for protein sequence encoding
- name: LIBSVM
  role: Consumes the encoded feature vectors downstream for RBF-kernel SVM classification
    after SVM-RFE+CBR feature selection
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:4320cd9fbdd783173dcbbe503a4d156edb2cb5b43e18a39ba3aa3019493cb735
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  - iFeature
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# protein-sequence-feature-vectorization

## Summary

Encode protein sequences into fixed-length numeric feature vectors using multiple descriptors from the iDBP-DEP pipeline: DDE (dipeptide composition), PSSM-DBT (Distance-Bigram Transformation, 1600-d), PSSM-DWT (Discrete Wavelet Transform, 1040-d), and NMBAC (Normalized Moreau-Broto Autocorrelation, 200-d), concatenated into a 3240-dimension vector per protein for DNA-binding protein prediction.

## When to use

Use when raw protein sequences (PDB1075/PDB594/PDB186 style FASTA) must be turned into fixed-length numeric descriptors before machine-learning classification, and PSI-BLAST PSSM profiles plus physicochemical properties are available.

## When NOT to use

- Input is already a per-protein feature matrix (e.g. the ~/Data/*/Feature Data .mat files) — re-vectorization would duplicate the encoding step
- No PSI-BLAST PSSM profile can be generated (e.g. no nr database access) — PSSM-DBT and PSSM-DWT descriptors cannot be computed, only DDE and NMBAC apply
- The downstream task requires residue-level rather than sequence-level fixed-length representations

## Inputs

- Protein FASTA sequences from benchmark datasets PDB1075 (525 DBPs / 550 non-DBPs), PDB594 (297/297), PDB186 (93/93)
- PSI-BLAST PSSM text profiles (N x 20 matrices) generated with 3 iterations and e-value 0.001 against the nr database
- Amino acid physicochemical property values (hydrophobicity, solvent accessibility, side-chain volume, polarity, polarizability, net charge index) from the AAindex Database (phy_6.mat)

## Outputs

- Per-protein DDE feature matrix (DDE.mat / DDE594.mat)
- Per-protein PSSM-DBT 1600-dimension feature matrix (PSSM_DBT_workspace.mat, PSSM_DBT_594.mat)
- Per-protein PSSM-DWT 1040-dimension feature matrix (PSSM_DWT_workspace.mat)
- Per-protein NMBAC 200-dimension feature matrix (feature_NMBAC_workspace.mat)
- Combined 3240-dimension multi-view feature vector per protein (DDE_DBT_DWT_NMBAC workspace .mat files)

## How to apply

Generate each protein's N x 20 PSSM with PSI-BLAST against the nr database (3 iterations, e-value 0.001) and parse the text profiles with Read_Text_files_PSSM.m. Compute four descriptors: DDE for dipeptide composition (DDE.py); PSSM-DBT, which for distances t = 0..3 computes bigram occurrence frequencies I(Rx, Ry | t) = (1/(N-t)) * sum_i P[i,x] * P[i+t,y] over all 400 amino acid pairs, concatenated into 1600 features (T = 3, chosen by 10-fold cross validation); PSSM-DWT, treating the PSSM as 20 discrete length-N signals decomposed 4 times, extracting max/min/mean/std of approximation and detail coefficients plus top-5 DCT coefficients of the approximation (52 features x 20 signals = 1040); and NMBAC, computing normalized Moreau-Broto autocorrelations for six physicochemical properties from the AAindex Database at lags d = 1..30 (nlag = 30) plus 20 amino acid frequencies (200-d). Concatenate the four blocks into the 3240-dimension feature vector and serialize each descriptor as a MATLAB .mat workspace file (e.g. PSSM_DBT_workspace.mat, feature_NMBAC_workspace.mat) before SVM-RFE+CBR feature selection and RBF-kernel LIBSVM classification (C = 1, gamma = 0.03125).

## Related tools

- **PSI-BLAST** (Generates the N x 20 PSSM evolutionary profiles against nr (3 iterations, e-value 0.001) that are the substrate for the PSSM-DBT and PSSM-DWT descriptors)
- **iFeature** (Referenced feature-generation toolkit context for protein sequence encoding)
- **LIBSVM** (Consumes the encoded feature vectors downstream for RBF-kernel SVM classification after SVM-RFE+CBR feature selection)

## Examples

```
matlab -batch "run('Code/Read_Text_files_PSSM.m'); run('Code/feature_Extract/PSSM-DBT/ExDBT.m')"
```

## Evaluation signals

- Each protein yields exactly 1600 PSSM-DBT features (4 distances x 400 amino acid pairs, T = 3)
- Each protein yields exactly 1040 PSSM-DWT features (20 amino acid signals x 52 features) and 200 NMBAC features (180 autocorrelation terms at nlag = 30 plus 20 amino acid frequencies)
- Full concatenated vector is 3240-dimension per protein (DDE + 1600 + 1040 + 200)
- PSSM-DBT remains the strongest individual descriptor, reproducing the reported post-feature-selection Jackknife Acc of 82.70% and MCC of 0.654 on PDB1075
- Serialized .mat workspace files match the README structure (e.g. PSSM_DBT_workspace.mat, feature_NMBAC.mat in ~/PDB1075_workspace)

## Limitations

- PSSM profiles depend on PSI-BLAST search against the nr database, and no version numbers are stated for LIBSVM or PSI-BLAST, so exact reproducibility may vary
- T (PSSM-DBT) = 3 and nlag (NMBAC) = 30 were chosen via 10-fold cross validation on these benchmarks and may not transfer to other datasets
- The descriptors are sequence-level encodings tuned for distinguishing DBPs from non-DBPs, not for residue-level binding site prediction

## Evidence

- [intro] we combined them with PSSM-DWT (1040-dimension) and NMBAC (200-dimension) descriptor to obtain 3240-dimension feature vector: [evidence span withheld — anchor status: not_found]
