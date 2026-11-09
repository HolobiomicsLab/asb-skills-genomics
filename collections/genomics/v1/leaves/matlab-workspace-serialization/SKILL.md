---
name: matlab-workspace-serialization
description: Use when computed per-sequence feature vectors (DDE, PSSM-DBT, PSSM-DWT,
  NMBAC) must be persisted as MATLAB workspace .mat files matching a published repository
  layout for later SVM training or result validation.
when_to_use_negative:
- The downstream consumer is not MATLAB/LIBSVM and can accept CSV/NumPy arrays directly,
  making .mat serialization unnecessary overhead
- Sequences are shorter than 50 residues or contain ambiguous residue 'X', which the
  iDBP-DEP preprocessing explicitly excludes
- You only need raw PSSM profiles rather than aggregated per-protein descriptor matrices
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: iDBP-DEP
  role: Repository providing DDE.py (Python feature extraction) and MATLAB scripts
    whose outputs are serialized into per-dataset workspace .mat files
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: LIBSVM
  role: Downstream consumer of the serialized workspace .mat feature data for RBF-kernel
    SVM training
- name: iLearn
  role: Cited toolkit that also integrates feature extraction and analysis for protein
    sequences as an alternative to the custom MATLAB serialization workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:7e1f9f26973eb30ff788d54811c35a4f57474844787df069bf8e47ddaa422b5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - iDBP-DEP
  - iLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# matlab-workspace-serialization

## Summary

Serialize per-protein feature matrices (e.g. 400-dim DDE, PSSM-DBT, PSSM-DWT, NMBAC vectors from the iDBP-DEP pipeline) into MATLAB workspace .mat files organized per benchmark dataset (PDB1075, PDB594, PDB186) so downstream SVM training and reproduction of deposited results is possible.

## When to use

Use when computed per-sequence feature vectors (DDE, PSSM-DBT, PSSM-DWT, NMBAC) must be persisted as MATLAB workspace .mat files matching a published repository layout for later SVM training or result validation.

## When NOT to use

- The downstream consumer is not MATLAB/LIBSVM and can accept CSV/NumPy arrays directly, making .mat serialization unnecessary overhead
- Sequences are shorter than 50 residues or contain ambiguous residue 'X', which the iDBP-DEP preprocessing explicitly excludes
- You only need raw PSSM profiles rather than aggregated per-protein descriptor matrices

## Inputs

- FASTA protein sequences from PDB1075, PDB594, PDB186 (sequences shorter than 50 residues or containing 'X' excluded)
- Per-protein 400-dimension DDE feature vectors
- Per-protein PSSM-DBT, PSSM-DWT (1040-dim), and NMBAC (200-dim) feature vectors
- PSSM profiles from ~/Data/<dataset>/<dataset>_seqs_PSSM

## Outputs

- DDE_workspace.mat / DDE594_workspace.mat / DDE186.mat (400-dim feature matrices)
- PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat
- Combined DDE_DBT_DWT_NMBAC_594_workspace.mat (3240-dim features)
- Per-dataset feature matrices under ~/Data/<dataset>/Feature Data as .mat files

## How to apply

After computing per-protein descriptors (e.g. the 400-dimension DDE vector via Dc(r,s)=N_rs/(N-1), Tm from 61 sense codon counts, and Tv=Tm(1-Tm)/(N-1)), assemble the vectors into a per-dataset feature matrix. Serialize the matrix as a MATLAB workspace file and place it in the dataset-specific workspace directory, mirroring the iDBP-DEP layout: DDE_workspace.mat, DDE594_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat, and the combined DDE_DBT_DWT_NMBAC_594_workspace.mat. Keep the raw sequences and PSSM profiles under ~/Data/<dataset>/ separate from the workspace outputs. The rationale is reproducibility: the deposited .mat files are the exact training inputs for the LIBSVM RBF-kernel model (C=1, gamma=0.03125) after SVM-RFE+CBR feature selection, so serialized workspaces must match them dimension-for-dimension.

## Related tools

- **iDBP-DEP** (Repository providing DDE.py (Python feature extraction) and MATLAB scripts whose outputs are serialized into per-dataset workspace .mat files) — https://githup.com/Zll-codeside/iDBP-DEP
- **LIBSVM** (Downstream consumer of the serialized workspace .mat feature data for RBF-kernel SVM training)
- **iLearn** (Cited toolkit that also integrates feature extraction and analysis for protein sequences as an alternative to the custom MATLAB serialization workflow)

## Examples

```
python Code/feature_Extract/DDE/DDE.py  # then in MATLAB: save('PDB1075_workspace/DDE_workspace.mat','DDE_features')
```

## Evaluation signals

- Each protein in PDB1075, PDB594 and PDB186 yields exactly a 400-dimension DDE feature vector before serialization
- Serialized DDE workspace matrices match the deposited DDE_workspace.mat / DDE594_workspace.mat / DDE186.mat files in the iDBP-DEP repository
- Combined workspace (DDE + PSSM-DBT + PSSM-DWT + NMBAC) totals 3240 dimensions per protein (400 + 1600 + 1040 + 200)
- Workspace files are placed in dataset-specific directories (PDB1075_workspace, PDB594_workspace) with README-consistent naming
- Feature matrices loaded from .mat feed train_1075_demo.txt workflow and reproduce reported Jackknife/10-fold CV results

## Limitations

- No accession IDs or provenance are given for the benchmark datasets PDB1075, PDB594, PDB186 in the discussion, limiting provenance checks on serialized workspaces
- Sequences shorter than 50 residues or containing 'X' are excluded, so serialized matrices will not cover every FASTA entry in the raw datasets
- The repo's README contains a merge-conflict artifact (<<<<<<< HEAD / >>>>>>> iDEP), so file names may differ between the two HEADs (e.g. DDE.mat vs DDE_workspace.mat)

## Evidence

- [methods] we initiatively applied Dipeptide Deviation from Expected Mean (DDE) descriptor to represent the dipeptide composition.: [evidence span withheld — anchor status: not_found]
