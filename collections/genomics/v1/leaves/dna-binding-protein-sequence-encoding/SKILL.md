---
name: dna-binding-protein-sequence-encoding
description: Use when raw protein sequences (plus PSI-BLAST PSSM profiles) must be
  converted into discriminative numeric feature vectors to classify DBPs vs non-DBPs
  and reproduce iDBP-DEP-style prediction benchmarks.
when_to_use_negative:
- Feature matrices (.mat workspace files) are already computed and the task is only
  classifier training or independent testing — skip encoding and use the workspace
  directly
- Sequences lack PSI-BLAST evolutionary profiles (PSSM/PSFM), since PSSM-DBT, PSSM-DWT
  and part of the multi-view representation depend on them
- The task is residue-level binding site prediction rather than whole-sequence DBP
  vs non-DBP classification
edam_operation: http://edamontology.org/operation_0267
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: PSI-BLAST
  role: Generates the PSSM evolutionary profiles that feed the PSSM-DBT, PSSM-DWT
    and NMBAC encodings
- name: LIBSVM
  role: Implements the RBF-kernel SVM classifier trained on the selected feature subset
- name: iDBP-DEP
  role: Reference implementation supplying feature extraction code (DDE.py, BAC.m,
    getDBT.m, getDWT.m), workspaces and predictor scripts
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:dab6e5bffcf73d0200ce5461593da0be56830b96d74a2c2d8979a48d4b048a89
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  - iDBP-DEP
  edam_operation: http://edamontology.org/operation_0267
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# dna-binding-protein-sequence-encoding

## Summary

Encodes protein sequences into multi-view feature vectors for DNA-binding protein (DBP) prediction by combining DDE dipeptide composition, PSSM-DBT and PSSM-DWT evolutionary descriptors, and NMBAC physicochemical autocorrelation into a 3240-dimension feature space, followed by SVM-RFE+CBR feature selection and SVM classification.

## When to use

Use when raw protein sequences (plus PSI-BLAST PSSM profiles) must be converted into discriminative numeric feature vectors to classify DBPs vs non-DBPs and reproduce iDBP-DEP-style prediction benchmarks.

## When NOT to use

- Feature matrices (.mat workspace files) are already computed and the task is only classifier training or independent testing — skip encoding and use the workspace directly
- Sequences lack PSI-BLAST evolutionary profiles (PSSM/PSFM), since PSSM-DBT, PSSM-DWT and part of the multi-view representation depend on them
- The task is residue-level binding site prediction rather than whole-sequence DBP vs non-DBP classification

## Inputs

- Protein sequences in FASTA-like format (e.g. PDB1075: 525 DBPs / 550 non-DBPs; PDB594: 297/297; PDB186: 93/93)
- PSI-BLAST PSSM profiles for each sequence (e.g. PDB1075_seqs_PSSM, PDB186_seqs_PSSM directories)
- Physicochemical property matrix (phy_6.mat) for NMBAC computation

## Outputs

- Per-descriptor feature matrices as .mat workspace files (DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat)
- Combined 3240-dimension feature vector per sequence; after SVM-RFE+CBR, an optimal feature index subset (top 424 for PDB1075)
- Trained LIBSVM RBF model and predicted -1/+1 DBP labels with Acc, Sn, Sp, MCC, AUC metrics

## How to apply

Generate PSSM profiles by searching each target sequence against the nr database with PSI-BLAST. Encode each sequence with four descriptors: Dipeptide Deviation from Expected Mean (DDE), PSSM Distance-Bigram Transformation (PSSM-DBT with parameter T=3), 4-scale PSSM Discrete Wavelet Transformation (PSSM-DWT, computing max/min/mean/std of approximation and detail coefficients at each level), and Normalized Moreau-Broto Autocorrelation (NMBAC with nlag=30), concatenating them into a 3240-dimension vector. Apply SVM-RFE+CBR to remove noisy and redundant features and obtain the optimal subset (e.g. the top 424 features for PDB1075). Train an SVM with RBF kernel via LIBSVM (C=1, gamma=0.03125) and evaluate with Jackknife/cross-validation on PDB1075/PDB594 or independent testing on PDB186 using Acc, Sn, Sp, MCC and ROC AUC.

## Related tools

- **PSI-BLAST** (Generates the PSSM evolutionary profiles that feed the PSSM-DBT, PSSM-DWT and NMBAC encodings)
- **LIBSVM** (Implements the RBF-kernel SVM classifier trained on the selected feature subset)
- **iDBP-DEP** (Reference implementation supplying feature extraction code (DDE.py, BAC.m, getDBT.m, getDWT.m), workspaces and predictor scripts) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
matlab -r "load PDB1075_workspace/DDE_DBT_DWT_NMBAC_594_workspace.mat; train_1075"  % then run Independent_test.m on PDB186 workspace features
```

## Evaluation signals

- Encoded feature dimensionality is 3240 (DDE + PSSM-DBT + PSSM-DWT 1040-dim + NMBAC 200-dim) before feature selection
- After SVM-RFE+CBR, the composed top-424 feature index reproduces the paper's optimal subset for PDB1075
- Independent test on PDB186 with the trained SVM matches Table 8: Acc 80.1%, Sn 93.6%, Sp 66.7%, MCC 0.684, AUC 0.854
- Grid-searched SVM hyperparameters equal C=1 and gamma=0.03125
- Descriptor hyperparameters match the 10-fold cross-validated choices: T=3 (PSSM-DBT) and nlag=30 (NMBAC)

## Limitations

- No version numbers stated for LIBSVM or PSI-BLAST; README shows libsvm-3.21 but the article omits versions
- No accession IDs or provenance given for PDB1075, PDB594, PDB186 beyond their construction from PDB searches
- RBF kernel and hyperparameters (C=1, gamma=0.03125, top-424 features) were tuned for these benchmark datasets; they may not transfer to other sequence sets
- Jackknife test on PDB1075/PDB594 is computationally expensive relative to 10-fold cross-validation

## Evidence

- [intro] 3240-dimension combination: [evidence span withheld — anchor status: not_found]
