---
name: pssm-distance-bigram-feature-extraction
description: Use when starting from PSI-BLAST PSSM text profiles (N x 20 matrices,
  3 iterations, e-value 0.001 against nr) and you need position/evolution-aware features
  like PSSM-DBT for protein classification.
when_to_use_negative:
- Input is already an encoded feature table or a combined 3240-dimension DDE/PSSM-DBT/PSSM-DWT/NMBAC
  vector — no re-encoding is needed.
- You only have raw FASTA sequences and cannot run PSI-BLAST to obtain PSSM profiles;
  DBT operates on PSSM matrices, not on sequence strings directly.
- You need the other iDBP-DEP descriptors (DDE, PSSM-DWT, NMBAC) rather than the PSSM-DBT
  evolutionary block.
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: PSI-BLAST
  role: Generates the PSSM profiles (N x 20) against the non-redundant database (nr)
    that are the input to the DBT encoding
- name: getDBT.m / DBT186.m / ExDBT.m
  role: MATLAB scripts in Code/feature_Extract/PSSM-DBT that compute the Distance-Bigram
    Transformation features from parsed PSSMs
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: Read_Text_files_PSSM.m
  role: MATLAB loader that parses the PSI-BLAST PSSM text files into matrices before
    DBT encoding
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:0de5da6c79960c7811065055288929b3624da3098d759101fea85fa03c8cbf46
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PSI-BLAST
  - Read_Text_files_PSSM.m
  - getDBT.m / DBT186.m / ExDBT.m
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pssm-distance-bigram-feature-extraction

## Summary

Encodes PSI-BLAST PSSM evolutionary profiles of protein sequences with the Distance-Bigram Transformation (DBT), producing a 1600-dimension feature vector per protein that captures relative amino-acid position information and evolutionary information for DNA-binding protein prediction (iDBP-DEP).

## When to use

Use when starting from PSI-BLAST PSSM text profiles (N x 20 matrices, 3 iterations, e-value 0.001 against nr) and you need position/evolution-aware features like PSSM-DBT for protein classification.

## When NOT to use

- Input is already an encoded feature table or a combined 3240-dimension DDE/PSSM-DBT/PSSM-DWT/NMBAC vector — no re-encoding is needed.
- You only have raw FASTA sequences and cannot run PSI-BLAST to obtain PSSM profiles; DBT operates on PSSM matrices, not on sequence strings directly.
- You need the other iDBP-DEP descriptors (DDE, PSSM-DWT, NMBAC) rather than the PSSM-DBT evolutionary block.

## Inputs

- PSI-BLAST PSSM text profiles (N x 20 matrices, generated with 3 iterations and e-value 0.001 against nr), e.g. PDB1075_seqs_PSSM files
- Protein sequences from benchmark datasets PDB1075 (525 DBPs / 550 non-DBPs), PDB594, or PDB186

## Outputs

- 1600-dimension PSSM-DBT feature vector per protein (4 distance blocks x 400 amino-acid pairs)
- Encoded feature matrices saved as PSSM_DBT_workspace.mat, PSSM_DBT_594_workspace.mat, or PSSM_DBT_186.mat

## How to apply

Parse the PSSM text profiles with the MATLAB loader Read_Text_files_PSSM.m from the iDBP-DEP repository (PSSM text files live under ~/Data/PDB1075/PDB1075_seqs_PSSM etc.). For each distance t in {0, 1, 2, 3}, compute the bigram occurrence frequency I(Rx, Ry | t) = (1/(N-t)) * sum_i P[i,x] * P[i+t,y] for all 400 amino acid pairs (Rx, Ry), where t=0 corresponds to an amino acid itself; T is fixed to 3, as chosen in the paper by 10-fold cross validation. Concatenate the four 400-dimension bigram blocks into the 1600-dimension PSSM-DBT feature vector per protein. Save the encoded matrices as PSSM_DBT_workspace.mat (and PSSM_DBT_594_workspace.mat for PDB594, PSSM_DBT_186.mat for PDB186). The rationale is that DBT captures the relative position information of amino acids and evolutionary information, and PSSM-DBT was the strongest individual descriptor before SVM-RFE+CBR feature selection and SVM (LIBSVM, RBF, C=1, gamma=0.03125) classification.

## Related tools

- **PSI-BLAST** (Generates the PSSM profiles (N x 20) against the non-redundant database (nr) that are the input to the DBT encoding)
- **getDBT.m / DBT186.m / ExDBT.m** (MATLAB scripts in Code/feature_Extract/PSSM-DBT that compute the Distance-Bigram Transformation features from parsed PSSMs) — https://githup.com/Zll-codeside/iDBP-DEP
- **Read_Text_files_PSSM.m** (MATLAB loader that parses the PSI-BLAST PSSM text files into matrices before DBT encoding) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
matlab -batch "P = Read_Text_files_PSSM('Data/PDB1075/PDB1075_seqs_PSSM'); F = getDBT(P); save('PSSM_DBT_workspace.mat','F')"
```

## Evaluation signals

- Each protein yields exactly 1600 features (4 distances x 400 amino-acid pairs); any other dimension indicates a parsing or distance-range error.
- Distance parameter T is 3 (blocks for t=0,1,2,3), matching the 10-fold cross-validation choice reported in the paper.
- Encoded matrix files reproduce the repository workspace layout: PSSM_DBT_workspace.mat for PDB1075 and PSSM_DBT_594_workspace.mat for PDB594.
- Downstream reproduction check: PSSM-DBT is the strongest individual descriptor, with post-feature-selection Jackknife Acc of 82.70% and MCC of 0.654 on PDB1075.

## Limitations

- Requires PSI-BLAST PSSM profiles against nr; sequences without precomputed PSSMs must first be searched with 3 iterations and e-value 0.001.
- No version number for PSI-BLAST is stated in the paper, and LIBSVM version (3.21 in the repo) is only evident from the repository file layout.
- The paper does not provide accession IDs or provenance for PDB1075, PDB594, and PDB186, complicating provenance checks of the inputs.
- Performance figures (e.g. Acc 82.70%, MCC 0.654 on PDB1075) apply after SVM-RFE+CBR feature selection, not to the raw 1600-dimension DBT vector alone.

## Evidence
