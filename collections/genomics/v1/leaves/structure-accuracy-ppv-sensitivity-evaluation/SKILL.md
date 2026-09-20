---
name: structure-accuracy-ppv-sensitivity-evaluation
description: Use when you have a reactivity profile (e.g. nanoSHAPE or SHAPE-MaP)
  plus the same RNA folded without probing data, and need to quantify constrained-structure
  accuracy by base-pair set comparison (PPV/sensitivity) against the NPD reference.
when_to_use_negative:
- The goal is modification detection/localization from nanopore signal rather than
  judging secondary structure prediction accuracy — use tombo detect_modifications
  instead of base-pair set comparison.
- There is no reactivity profile available and no NPD reference fold, so neither constrained-vs-NPD
  comparison nor PPV/sensitivity can be computed.
- The reactivity profile has poor resolution (e.g. the 5' ends of long RNAs), so converting
  it to pseudo-free-energy constraints would mislead the fold rather than refine it.
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RNAstructure
  role: Folds the pri-miR-17~92 sequence under nanoSHAPE/SHAPE-MaP reactivities converted
    to pseudo-free-energy soft constraints to produce constrained MFE structures for
    PPV/sensitivity comparison
- name: RNAfold
  role: Generates constrained centroid structures using SHAPE-derived reactivity constraints
    (e.g. 'M'/'W' shape method) and NPD reference folds
- name: tombo
  role: Resquiggles single fast5 reads and runs model_sample_compare against unmodified
    controls to yield the modification/reactivity statistics underlying the nanoSHAPE
    profile being benchmarked
  repo: https://nanoporetech.github.io/tombo/
- name: extract_array.py
  role: Extracts per-read modification sites from fast5 files into a numpy array (.npy)
    for python processing of the reactivity signal used to build constraints
  repo: https://github.com/physnano/rRNA_nanoSHAPE
- name: python
  role: Processes the extracted reactivity arrays, computes PPV/sensitivity between
    base-pair sets, and writes primir17-92_ppv_sensitivity.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:5db07a5a0b5da381e00e4e9747e9daf1b82668f0ae3888e6700c719e89eb3a78
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
  - python
  - tombo
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# structure-accuracy-ppv-sensitivity-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark for judging whether probing-data-constrained RNA secondary structure predictions (nanoSHAPE, SHAPE-MaP) outperform unconstrained no-probing-data (NPD) folds, using positive predictive value (PPV), sensitivity, and recovery of known miRNA hairpins in pri-miR-17~92.

## When to use

Use when you have a reactivity profile (e.g. nanoSHAPE or SHAPE-MaP) plus the same RNA folded without probing data, and need to quantify constrained-structure accuracy by base-pair set comparison (PPV/sensitivity) against the NPD reference.

## When NOT to use

- The goal is modification detection/localization from nanopore signal rather than judging secondary structure prediction accuracy — use tombo detect_modifications instead of base-pair set comparison.
- There is no reactivity profile available and no NPD reference fold, so neither constrained-vs-NPD comparison nor PPV/sensitivity can be computed.
- The reactivity profile has poor resolution (e.g. the 5' ends of long RNAs), so converting it to pseudo-free-energy constraints would mislead the fold rather than refine it.

## Inputs

- nanoSHAPE reactivity profile for pri-miR-17~92 (from tombo model_sample_compare per-read stats processed via extract_array.py into a .npy array)
- SHAPE-MaP reactivity profile for pri-miR-17~92
- FASTA reference sequence of the pri-miR-17~92 cluster
- Annotation of the six expected miRNA hairpins in the cluster
- Unmodified (native/control) nanopore reads for the no-probing-data reference condition

## Outputs

- Constrained MFE secondary structure predictions (RNAstructure) for nanoSHAPE and SHAPE-MaP profiles
- Constrained centroid secondary structure predictions (RNAfold) for nanoSHAPE and SHAPE-MaP profiles
- NPD (no-probing-data) reference structures
- PPV and sensitivity per constrained model vs NPD reference, written to primir17-92_ppv_sensitivity.csv

## How to apply

Normalize the nanoSHAPE and SHAPE-MaP reactivity profiles for constraint conversion, then fold the pri-miR-17~92 sequence under each profile: soft SHAPE pseudo-free-energy constraints in RNAstructure for MFE prediction, and SHAPE-derived reactivity constraints ('M' or 'W' shape method) in RNAfold for centroid prediction. Fold the identical sequence with no experimental constraints to produce the NPD reference models. Compare base-pair sets of each constrained MFE/centroid structure against the NPD models to compute PPV and sensitivity per model. Decision point: accept a constrained model only when all six miRNA hairpins expected in the pri-miR-17~92 cluster are recovered and PPV/sensitivity are computable against the NPD reference; additionally verify that the nanoSHAPE-constrained ensemble shifts toward the SHAPE-MaP-informed model. Write the per-model PPV/sensitivity metrics to primir17-92_ppv_sensitivity.csv.

## Related tools

- **RNAstructure** (Folds the pri-miR-17~92 sequence under nanoSHAPE/SHAPE-MaP reactivities converted to pseudo-free-energy soft constraints to produce constrained MFE structures for PPV/sensitivity comparison)
- **RNAfold** (Generates constrained centroid structures using SHAPE-derived reactivity constraints (e.g. 'M'/'W' shape method) and NPD reference folds)
- **tombo** (Resquiggles single fast5 reads and runs model_sample_compare against unmodified controls to yield the modification/reactivity statistics underlying the nanoSHAPE profile being benchmarked) — https://nanoporetech.github.io/tombo/
- **extract_array.py** (Extracts per-read modification sites from fast5 files into a numpy array (.npy) for python processing of the reactivity signal used to build constraints) — https://github.com/physnano/rRNA_nanoSHAPE
- **python** (Processes the extracted reactivity arrays, computes PPV/sensitivity between base-pair sets, and writes primir17-92_ppv_sensitivity.csv)

## Examples

```
python extract_array.py --fa primir17-92.fa --f5 single_fast5_MODIFIED --out reactivity.npy --rna-name pri-miR-17~92 --sort-reads --current; RNAstructure Fold primir17-92.fa --shape reactivity.txt
```

## Evaluation signals

- Both SHAPE-MaP-constrained and nanoSHAPE-constrained centroid predictions recover all six miRNA hairpins expected in the pri-miR-17~92 cluster.
- PPV and sensitivity are computable for each constrained MFE/centroid structure against the NPD reference base-pair sets.
- The nanoSHAPE-constrained ensemble shifts toward the SHAPE-MaP-informed model rather than diverging from it.
- primir17-92_ppv_sensitivity.csv is produced with per-model PPV/sensitivity rows covering all constrained and NPD models.
- Constrained folds were generated only after reactivities were normalized for constraint conversion, using soft constraints (RNAstructure) or 'M'/'W' shape methods (RNAfold).

## Limitations

- nanoSHAPE has poor resolution of reactivity profiles at the 5' ends of longer RNA molecules, so constraints in those regions are unreliable.
- High rates of AcIm modification lower the number of full-length reads, overall yield, and alignment rates, thinning the data behind the reactivity profile.
- The nanoSHAPE workflow requires comparison against an unmodified (control) RNA sample; tombo model_sample_compare calls typically use around 100 minimum test reads for in vitro samples.
- Signal discrimination in direct RNA nanopore sequencing depends on kmer sequence context and raw-current alignment quality, affecting signal fidelity upstream of any structure benchmark.

## Evidence
