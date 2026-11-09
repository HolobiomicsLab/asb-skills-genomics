---
name: amino-acid-bigram-frequency-computation
description: Use when protein sequences already have PSI-BLAST PSSM profiles (N x
  20 text matrices) and you need position/evolution-aware residue-pair features, e.g.
  for DNA-binding protein (DBP) classifiers like iDBP-DEP.
when_to_use_negative:
- Input is already a computed feature table (e.g. DDE.mat, feature_NMBAC.mat, PSSM_DWT.mat)
  rather than raw PSSM text profiles — recomputing DBT is redundant
- Sequences lack precomputed PSSM profiles and you only need composition or physicochemical
  descriptors — use DDE or NMBAC on the raw sequence instead of running PSI-BLAST
  just for DBT
- You need position-unaware evolutionary summary features (e.g. average PSSM scores)
  where the 1600-dim DBT vector's position sensitivity adds no value
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: PSI-BLAST
  role: Generates the PSSM profiles (searched against the nr database) that the DBT
    bigram computation consumes as input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:429099e6c6802ab6b3e316676bb8eb658ecf004d1dadc996ead269766fe0413a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PSI-BLAST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Amino-acid bigram frequency computation

## Summary

Computes amino-acid bigram occurrence frequencies over PSI-BLAST PSSM profiles using the Distance-Bigram Transformation (DBT) algorithm from the iDBP-DEP paper, capturing relative amino-acid position information and evolutionary information as a 1600-dimension feature vector (4 distances x 400 residue pairs).

## When to use

Use when protein sequences already have PSI-BLAST PSSM profiles (N x 20 text matrices) and you need position/evolution-aware residue-pair features, e.g. for DNA-binding protein (DBP) classifiers like iDBP-DEP.

## When NOT to use

- Input is already a computed feature table (e.g. DDE.mat, feature_NMBAC.mat, PSSM_DWT.mat) rather than raw PSSM text profiles — recomputing DBT is redundant
- Sequences lack precomputed PSSM profiles and you only need composition or physicochemical descriptors — use DDE or NMBAC on the raw sequence instead of running PSI-BLAST just for DBT
- You need position-unaware evolutionary summary features (e.g. average PSSM scores) where the 1600-dim DBT vector's position sensitivity adds no value

## Inputs

- PSI-BLAST PSSM text profile per protein (N x 20 matrix, generated with 3 iterations, e-value 0.001, against the nr database)
- Protein sequence datasets PDB1075 (525 DBPs / 550 non-DBPs), PDB594 (297/297), PDB186 (93/93)

## Outputs

- 1600-dimension PSSM-DBT feature vector per protein (4 distance blocks x 400 residue-pair bigrams)
- Encoded per-dataset feature matrices saved as MATLAB .mat files, e.g. PSSM_DBT_workspace.mat, PSSM_DBT_594_workspace.mat, PSSM_DBT_186.mat

## How to apply

First parse the PSI-BLAST PSSM text profiles (N x 20 matrices generated with 3 iterations and e-value 0.001 against nr) using the Read_Text_files_PSSM.m loader. For each distance t in {0, 1, 2, 3}, compute the bigram occurrence frequency I(Rx, Ry | t) = (1/(N-t)) * sum_i P[i,x] * P[i+t,y] over the PSSM values for all 400 amino acid pairs (Rx, Ry), where t=0 corresponds to an amino acid itself. Concatenate the four 400-dimension bigram blocks into the 1600-dimension PSSM-DBT feature vector per protein, with the maximum distance T set to 3 (chosen via 10-fold cross validation in the paper). Save the per-dataset encoded matrices as .mat workspaces (e.g. PSSM_DBT_workspace.mat for PDB1075, PSSM_DBT_594_workspace.mat for PDB594, PSSM_DBT_186.mat for PDB186) using the getDBT.m/DBT186.m/ExDBT.m scripts under Code/feature_Extract/PSSM-DBT. The rationale is that bigrams over the PSSM, unlike raw sequence bigrams, weight each residue pair by its evolutionary probability, so relative position and evolutionary information are captured jointly.

## Related tools

- **PSI-BLAST** (Generates the PSSM profiles (searched against the nr database) that the DBT bigram computation consumes as input)

## Examples

```
matlab -r "addpath('Code/feature_Extract/PSSM-DBT'); getDBT"
```

## Evaluation signals

- Each protein yields exactly 1600 features: 4 distances (t=0..3) x 400 amino-acid pairs
- Distance parameter T equals 3, as fixed by 10-fold cross validation in the paper
- Encoded matrices are stored with the expected filenames (PSSM_DBT_workspace.mat, PSSM_DBT_594_workspace.mat) in the workspace folders
- On PDB1075, PSSM-DBT is the strongest individual descriptor, reproducing the reported post-feature-selection Jackknife Acc of 82.70% and MCC of 0.654
- Parsed PSSM row counts match each protein length N so that normalization factors (1/(N-t)) are computed with the correct N

## Limitations

- Requires precomputed PSI-BLAST PSSM profiles; no version number for PSI-BLAST is stated in the paper, so PSSM reproduction may vary with PSI-BLAST/nr database versions
- T=3 is a dataset-tuned choice (10-fold CV); other tasks may need re-tuning of the maximum distance
- The 1600-dim vector is dense and was designed to be followed by SVM-RFE+CBR feature selection to remove noisy and redundant features before SVM classification

## Evidence

- [intro] Encode PSSM profile with Distance-Bigram Transformation to capture relative position and evolutionary information: "encoded PSSM profile with Distance-Bigram Transformation (DBT) algorithm to capture the relative position information of amino acids and evolutionary information effectively"
- [methods] PSSM replaces the original sequence for DBT feature extraction: "we used the PSSM replace the original protein sequence to extract the DBT features"
- [methods] T parameter for PSSM-DBT chosen by cross validation: "Thus, we chose T (PSSM-DBT) as 3 and nlag (NMBAC) as 30 in our study."
- [readme] README locates the PSSM inputs and feature-extraction code: "All the protein data and PSSM profile in ~/Data and Extract features code in file ~/Code/feature_Extract"
- [methods] PSI-BLAST generates the PSSM inputs: "The PSSM profiles[29] used in this paper were generated by searching for target protein in non-redundant database (nr) using PSI-BLAST.[15, 30]"
