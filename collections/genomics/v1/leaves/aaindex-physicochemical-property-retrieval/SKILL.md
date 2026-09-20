---
name: aaindex-physicochemical-property-retrieval
description: Use when encoding protein sequences with property-based autocorrelation
  descriptors (e.g. NMBAC) and you need the six AAindex property tables — hydrophobicity,
  solvent accessibility, side-chain volume, polarity, polarizability, net charge index
  — standardized per amino acid.
when_to_use_negative:
- Sequences contain non-standard amino acids — the property tables and descriptor
  are defined over the 20 natural amino acids only
- The descriptor computation is already done and feature_NMBAC_*.mat files are available
  for the target dataset (PDB186/PDB594/PDB1075)
- The goal is evolutionary-profile encoding (PSSM-DBT or PSSM-DWT), which uses PSI-BLAST
  PSSMs rather than AAindex property tables
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: iDBP-DEP NMBAC module (BAC.m, PYH_6.m, phy_6.mat, ExBAC_demo.txt)
  role: MATLAB scripts and property data (phy_6.mat) that supply the standardized
    physicochemical properties consumed by the BAC autocorrelation computation
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: iFeature
  role: Mentioned as an external toolkit that facilitates protein feature generation,
    an alternative route to property-based descriptors
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:2010adce1941718037e2f677639b50d449a2e07a1b425dbe4b34ffb40278b60b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PSI-BLAST
  - iDBP-DEP NMBAC module (BAC.m, PYH_6.m, phy_6.mat, ExBAC_demo.txt)
  - iFeature
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# aaindex-physicochemical-property-retrieval

## Summary

Retrieves amino acid physicochemical property value sets (six properties from the AAindex Database) and standardizes them across the 20 natural amino acids so they can drive autocorrelation descriptors such as the 200-dimension NMBAC used in iDBP-DEP.

## When to use

Use when encoding protein sequences with property-based autocorrelation descriptors (e.g. NMBAC) and you need the six AAindex property tables — hydrophobicity, solvent accessibility, side-chain volume, polarity, polarizability, net charge index — standardized per amino acid.

## When NOT to use

- Sequences contain non-standard amino acids — the property tables and descriptor are defined over the 20 natural amino acids only
- The descriptor computation is already done and feature_NMBAC_*.mat files are available for the target dataset (PDB186/PDB594/PDB1075)
- The goal is evolutionary-profile encoding (PSSM-DBT or PSSM-DWT), which uses PSI-BLAST PSSMs rather than AAindex property tables

## Inputs

- Amino acid physicochemical property value sets from the AAindex Database (six properties: hydrophobicity, solvent-accessible surface area, side-chain volume, polarity, polarizability, net charge index), supplied as phy_6.mat
- Protein sequences from benchmark datasets (PDB1075, PDB594, PDB186)

## Outputs

- Standardized (mean/SD) six-property tables over the 20 natural amino acids
- Per-protein 200-dimension NMBAC feature vectors (180 autocorrelation terms at nlag = 30 plus 20 amino acid frequencies)
- Assembled feature matrix saved as feature_NMBAC_workspace.mat (and dataset-specific variants e.g. feature_NMBAC_594.mat)

## How to apply

Load the six amino acid physicochemical property value sets from the AAindex Database, supplied in the iDBP-DEP repository as phy_6.mat next to the NMBAC module (BAC.m, PYH_6.m, ExBAC_demo.txt under ~/Code/feature_Extract/NMBAC). Centralize and standardize each property set across the 20 amino acids using its mean and standard deviation before any sequence-level computation. The standardized properties then feed the Normalized Moreau-Broto Autocorrelation at lags d = 1 to nlag = 30 (chosen via 10-fold cross validation in the paper), producing 6 x 30 = 180 autocorrelation terms per protein; appending the 20 amino acid frequencies yields the 200-dimension NMBAC vector saved as feature_NMBAC_workspace.mat. Use ExBAC_demo.txt as the worked example of the property-to-autocorrelation flow.

## Related tools

- **iDBP-DEP NMBAC module (BAC.m, PYH_6.m, phy_6.mat, ExBAC_demo.txt)** (MATLAB scripts and property data (phy_6.mat) that supply the standardized physicochemical properties consumed by the BAC autocorrelation computation) — https://githup.com/Zll-codeside/iDBP-DEP
- **iFeature** (Mentioned as an external toolkit that facilitates protein feature generation, an alternative route to property-based descriptors)

## Examples

```
matlab -batch "run('Code/feature_Extract/NMBAC/PYH_6.m')"  % loads phy_6.mat properties and mirrors ExBAC_demo.txt to compute BAC features
```

## Evaluation signals

- Every protein yields exactly a 200-dimension vector: 6 properties x 30 lags = 180 autocorrelation terms plus 20 amino acid frequencies
- nlag is fixed at 30, matching the paper's 10-fold cross-validation selection ('nlag (NMBAC) as 30')
- Each of the six property sets is standardized (mean/SD) across all 20 amino acids before autocorrelation computation
- Output matrix dimensions match the released workspace files (feature_NMBAC_workspace.mat, feature_NMBAC_594.mat, feature_NMBAC_186.mat)
- The 180 property terms come only from the six named AAindex properties; any additional or missing property indicates a wrong phy_6.mat mapping

## Limitations

- nlag = 30 was selected by 10-fold cross validation on the PDB1075/PDB594 benchmarks; other prediction tasks may need re-tuning
- Descriptor covers only the 20 natural amino acids, so non-standard residues break the property lookup
- The README gives no software version numbers or provenance for the property tables beyond phy_6.mat itself

## Evidence

- [methods] NMBAC descriptor defined on AAindex properties: "Normalized Moreau-Broto Autocorrelation(NMBAC)[35] descriptor is defined based on the amino acid physicochemical properties of a protein sequence."
- [methods] nlag chosen by cross-validation: "Thus, we chose T (PSSM-DBT) as 3 and nlag (NMBAC) as 30 in our study."
- [readme] NMBAC module files and property data in repo: "All the protein data and PSSM profile in ~/Data and Extract features code in file ~/Code/feature_Extract"
- [abstract] physicochemical properties as one multi-view feature source: "performs DNA-binding prediction by using the discriminative feature derived from multi-view feature sources including evolutionary profile, dipeptide composition, and physicochemical properties with"
