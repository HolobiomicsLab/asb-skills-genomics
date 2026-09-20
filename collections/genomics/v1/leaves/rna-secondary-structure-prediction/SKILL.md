---
name: rna-secondary-structure-prediction
description: Use when single-molecule SHAPE reactivity profiles (nanoSHAPE or SHAPE-MaP)
  for a specific RNA are available and the goal is to test whether constrained folding
  recovers expected structural elements (e.g., miRNA hairpins).
when_to_use_negative:
- Input is already a folded structure model or base-pair set — use structure comparison/PPV
  tools directly instead of refolding
- No experimental probing data and no unmodified control exists — a nanoSHAPE/SHAPE-MaP
  sample-compare constraint cannot be derived; an NPD fold alone is the only option
- The target is a short RNA far shorter than needed for nanopore reads — short-read
  SHAPE-MaP approaches may be sufficient and simpler
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0102
tools:
- name: tombo
  role: Resquiggles raw nanopore signal and detects per-nucleotide modification statistics
    used to derive nanoSHAPE reactivity constraints
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: extract_array.py
  role: Converts tombo per-read statistics into a numpy array of modification sites
    for downstream processing
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: python
  role: Runs extract_array.py and downstream per-read modification visualization/plotting
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: RNAstructure
  role: Folds the RNA under nanoSHAPE soft pseudo-free-energy constraints to produce
    the constrained MFE structure
- name: RNAfold
  role: Generates centroid structures with SHAPE-MaP reactivity constraints ('M'/'W'
    shape methods) for comparison
- name: multi_to_single_fast5
  role: Prepares single-read fast5 files from multi-fast5 files so tombo can process
    reads individually
  repo: github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:0606d82df8a96a76f7e796518a381b7a342935367680c85c2a8c2518f32953a1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - RNAfold
  - RNAstructure
  - extract_array.py
  - multi_to_single_fast5
  - python
  - tombo
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rna-secondary-structure-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate secondary structure models for a target RNA by folding its sequence with SHAPE-derived pseudo-free energy constraints — comparing nanoSHAPE- and SHAPE-MaP-constrained predictions against no-probing-data (NPD) folds to quantify how probing data shifts the structural ensemble.

## When to use

Use when single-molecule SHAPE reactivity profiles (nanoSHAPE or SHAPE-MaP) for a specific RNA are available and the goal is to test whether constrained folding recovers expected structural elements (e.g., miRNA hairpins).

## When NOT to use

- Input is already a folded structure model or base-pair set — use structure comparison/PPV tools directly instead of refolding
- No experimental probing data and no unmodified control exists — a nanoSHAPE/SHAPE-MaP sample-compare constraint cannot be derived; an NPD fold alone is the only option
- The target is a short RNA far shorter than needed for nanopore reads — short-read SHAPE-MaP approaches may be sufficient and simpler

## Inputs

- Single-read fast5 files for modified (AcIm-treated) and unmodified control RNA
- FASTA reference containing the target RNA sequence (e.g., pri-miR-17~92)
- nanoSHAPE and/or SHAPE-MaP reactivity profiles normalized for constraint conversion
- tombo per-read statistics files from model_sample_compare

## Outputs

- Constrained MFE (RNAstructure) and centroid (RNAfold) secondary structure predictions
- No-probing-data (NPD) reference structure
- PPV and sensitivity metrics per model (primir17-92_ppv_sensitivity.csv)
- numpy array (.npy) of per-read modification statistics from extract_array.py

## How to apply

First, extract normalized reactivity profiles for the target RNA from nanopore data: convert multi-fast5 to single-read fast5 (multi_to_single_fast5), resquiggle with tombo, run tombo detect_modifications model_sample_compare against an unmodified control (minimum-test-reads ~100 for in vitro), then use extract_array.py to produce a numpy array of per-read current/dwell statistics. Next, convert reactivities into pseudo-free energy constraints: apply RNAstructure soft constraints for the nanoSHAPE profile and the SHAPE-derived reactivity constraint methods ('M' or 'W' via RNAfold) for the SHAPE-MaP profile. Fold the RNA under each constraint set to obtain an MFE structure (RNAstructure) and centroid structure (RNAfold), and generate an NPD reference by folding the identical sequence without probing constraints. Compare base-pair sets of each constrained model against the NPD reference to compute PPV and sensitivity, and inspect whether known functional elements (e.g., all six miRNA hairpins of pri-miR-17~92) are recovered.

## Related tools

- **tombo** (Resquiggles raw nanopore signal and detects per-nucleotide modification statistics used to derive nanoSHAPE reactivity constraints) — github.com/physnano/rRNA_nanoSHAPE
- **extract_array.py** (Converts tombo per-read statistics into a numpy array of modification sites for downstream processing) — github.com/physnano/rRNA_nanoSHAPE
- **python** (Runs extract_array.py and downstream per-read modification visualization/plotting) — github.com/physnano/rRNA_nanoSHAPE
- **RNAstructure** (Folds the RNA under nanoSHAPE soft pseudo-free-energy constraints to produce the constrained MFE structure)
- **RNAfold** (Generates centroid structures with SHAPE-MaP reactivity constraints ('M'/'W' shape methods) for comparison)
- **multi_to_single_fast5** (Prepares single-read fast5 files from multi-fast5 files so tombo can process reads individually) — github.com/physnano/rRNA_nanoSHAPE

## Examples

```
python extract_array.py --fa ref.fa --f5 modified_fast5/ --out reactivities.npy --rna-name pri-miR-17~92 --sort-reads --current
```

## Evaluation signals

- All six miRNA hairpins expected in the pri-miR-17~92 cluster are recovered by both SHAPE-MaP- and nanoSHAPE-constrained centroid predictions
- PPV and sensitivity values are computable for each constrained model against the NPD reference base-pair sets
- The nanoSHAPE-constrained ensemble shifts toward the SHAPE-MaP-informed model (constrained structures converge)
- model_sample_compare ran with --minimum-test-reads (typically 100 for in vitro) so per-nucleotide statistics are supported by sufficient reads
- Metrics per model are written to primir17-92_ppv_sensitivity.csv

## Limitations

- Electrophilic SHAPE reagents such as AcIm covalently modify the 3'-hydroxyl, preventing adaptor ligation; chemical treatment to create a terminal phosphate plus phosphatase and poly(A) tailing is required before sequencing
- High rates of AcIm modification lower the number of full-length reads, overall yield, and alignment rates
- nanoSHAPE has poor reactivity resolution at the 5' ends of longer RNA molecules
- Signal discrimination is a function of kmer context and raw current alignment; training sets for de novo modification identification without an IVT control do not exist

## Evidence
