---
name: shape-reactivity-constrained-rna-folding
description: Use when you have a normalized per-nucleotide SHAPE/SHAPE-MaP reactivity
  profile for an RNA and need a secondary structure model that incorporates that probing
  data, plus a benchmark of whether constraints improve folding.
when_to_use_negative:
- No experimental reactivity data exist for the target RNA — a no-probing-data fold
  is then the baseline, not the constrained model
- The goal is de novo modification detection on raw nanopore signal rather than structure
  modeling from already-derived reactivities
- Reactivities are unreliable (e.g., the article notes poor nanoSHAPE resolution at
  the 5' ends of long RNAs); constraints derived from such regions can mislead the
  fold
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0080
tools:
- name: RNAstructure
  role: Folds the RNA with nanoSHAPE reactivities applied as pseudo-free-energy soft
    constraints to produce the constrained MFE structure
- name: RNAfold
  role: Generates the centroid structure using SHAPE-MaP reactivity constraints (e.g.,
    'M'/'W' shape methods)
- name: tombo
  role: Resquiggles reads and detects modifications (model_sample_compare) to produce
    the per-read statistics underlying the nanoSHAPE reactivity profile
  repo: https://nanoporetech.github.io/tombo/
- name: extract_array.py
  role: Converts tombo per-read fast5 statistics into a numpy reactivity array consumable
    in python for constraint normalization
  repo: https://github.com/physnano/rRNA_nanoSHAPE
- name: python
  role: Drives normalization of reactivity arrays and downstream PPV/sensitivity computation
    against NPD models
  repo: https://github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:acf3b6115bb05799b71892b83886a26e0435e3091a878b9d9dc07f2bb27afb30
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# shape-reactivity-constrained-rna-folding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts SHAPE reactivity profiles (e.g., nanoSHAPE-derived from nanopore direct RNA sequencing, or SHAPE-MaP) into pseudo-free-energy soft constraints to fold an RNA with RNAstructure or RNAfold, and benchmarks the constrained models against no-probing-data (NPD) folds by PPV/sensitivity and recovery of expected structural motifs.

## When to use

Use when you have a normalized per-nucleotide SHAPE/SHAPE-MaP reactivity profile for an RNA and need a secondary structure model that incorporates that probing data, plus a benchmark of whether constraints improve folding.

## When NOT to use

- No experimental reactivity data exist for the target RNA — a no-probing-data fold is then the baseline, not the constrained model
- The goal is de novo modification detection on raw nanopore signal rather than structure modeling from already-derived reactivities
- Reactivities are unreliable (e.g., the article notes poor nanoSHAPE resolution at the 5' ends of long RNAs); constraints derived from such regions can mislead the fold

## Inputs

- Normalized per-nucleotide reactivity profile from nanoSHAPE (numpy .npy array output of extract_array.py, sorted by --current, --dwell, or --stdev)
- SHAPE-MaP reactivity profile for the same RNA
- FASTA reference sequence of the target RNA (e.g., pri-miR-17~92 cluster)

## Outputs

- Constrained MFE secondary structure prediction from RNAstructure (nanoSHAPE soft constraints)
- Constrained centroid structure prediction from RNAfold (SHAPE-MaP constraints)
- No-probing-data (NPD) reference structure for the same sequence
- PPV/sensitivity metrics per model vs the NPD reference (e.g., primir17-92_ppv_sensitivity.csv) and confirmation of recovered miRNA hairpins

## How to apply

First normalize the reactivity profiles (nanoSHAPE arrays produced by tombo resquiggle + detect_modifications model_sample_compare + extract_array.py, or SHAPE-MaP reactivities) for constraint conversion. Feed the nanoSHAPE profile to RNAstructure as soft pseudo-free-energy constraints for an MFE prediction, and supply SHAPE-MaP reactivities to RNAfold with the SHAPE-derived constraint method (e.g., 'M'/'W' shape methods) to generate a centroid structure. Fold the identical sequence with no probing data (NPD) as the reference model. Compute PPV and sensitivity of each constrained structure against the NPD base-pair sets, and verify the constrained models recover all biologically expected motifs — in the validation case, all six miRNA hairpins of the pri-miR-17~92 cluster written to primir17-92_ppv_sensitivity.csv. Accept the constrained models only when all expected hairpins are recovered and PPV/sensitivity values are computable, demonstrating the nanoSHAPE ensemble shifts toward the SHAPE-MaP-informed model.

## Related tools

- **RNAstructure** (Folds the RNA with nanoSHAPE reactivities applied as pseudo-free-energy soft constraints to produce the constrained MFE structure)
- **RNAfold** (Generates the centroid structure using SHAPE-MaP reactivity constraints (e.g., 'M'/'W' shape methods))
- **tombo** (Resquiggles reads and detects modifications (model_sample_compare) to produce the per-read statistics underlying the nanoSHAPE reactivity profile) — https://nanoporetech.github.io/tombo/
- **extract_array.py** (Converts tombo per-read fast5 statistics into a numpy reactivity array consumable in python for constraint normalization) — https://github.com/physnano/rRNA_nanoSHAPE
- **python** (Drives normalization of reactivity arrays and downstream PPV/sensitivity computation against NPD models) — https://github.com/physnano/rRNA_nanoSHAPE

## Examples

```
python extract_array.py --fa ref.fasta --f5 MODIFIED/ --out nanoSHAPE.npy --rna-name pri-miR-17~92 --sort-reads --current
```

## Evaluation signals

- All six expected miRNA hairpins in the pri-miR-17~92 cluster are recovered in both the SHAPE-MaP- and nanoSHAPE-constrained predictions
- PPV and sensitivity of each constrained MFE/centroid structure are computable against the NPD base-pair sets (no empty or mismatched base-pair comparisons)
- The nanoSHAPE-constrained ensemble shifts toward the SHAPE-MaP-informed model rather than resembling the unconstrained fold
- Reactivity inputs are properly normalized (e.g., ~100 minimum test reads for in vitro tombo sample-compare statistics) before constraint conversion

## Limitations

- nanoSHAPE reactivity resolution is poor at the 5' ends of longer RNA molecules, limiting constraint quality there
- High rates of AcIm (acetylimidazole) modification lower the number of full-length reads, overall yield, and alignment rates, degrading the reactivity profile
- Constraints are judged against the NPD model rather than a crystallographic truth, so PPV/sensitivity reflect ensemble perturbation, not absolute structural accuracy

## Evidence

- [other] Both SHAPE-MaP-constrained and nanoSHAPE-constrained centroid structure predictions recover all six miRNA hairpins expected in the pri-miR-17~92 cluster, confirming nanoSHAPE supports accurate RNA secondary structure modeling.: [evidence span withheld — anchor status: not_found]
