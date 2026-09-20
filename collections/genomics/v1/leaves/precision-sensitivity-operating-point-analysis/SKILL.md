---
name: precision-sensitivity-operating-point-analysis
description: Use when you have an unfiltered NanoSV SV call set from long-read (MinION/nanopore)
  data plus benchmark SV sites (1KG or Pacific Biosciences) and need to select a thresholded
  filter operating point trading precision against sensitivity.
when_to_use_negative:
- The SV call set is already filtered to a fixed operating point, so no threshold
  sweep is possible
- No benchmark SV catalog (1KG or Pacific Biosciences) is available to assign TP/FP
  labels to training calls
- The goal is SNV/indel variant calling from short-read Illumina data rather than
  long-read SV post-calling filtering
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: NanoSV
  role: Generates the raw SV call set and per-call fields (VAF, PID, MAPQ, coverage)
    from which the 12 training features are extracted
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Maps the nanopore reads to the reference; the README notes LAST alignments
    give the most accurate results for SV calling with NanoSV
  repo: http://last.cbrc.jp/
- name: sambamba
  role: Converts and sorts the LAST MAF-derived SAM into the sorted BAM required as
    NanoSV input
  repo: https://github.com/biod/sambamba
- name: scikit-learn
  role: Random forest classifier trained on the 12 labeled NanoSV features; threshold
    sweep over predicted scores locates the operating point
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:da73b5bbe3cfe08d3ee7b6e3a042098c0cf2c0ea63aa971515b7c9f68f89d817
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - LAST
  - NanoSV
  - sambamba
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# precision-sensitivity-operating-point-analysis

## Summary

Trains and evaluates a random-forest post-calling filter for NanoSV structural-variant calls, sweeping score thresholds to locate an operating point with high precision (~82%) and estimated sensitivity (~75%), mirroring the paper's extremely stringent filtering strategy on NA12878 nanopore SV calls.

## When to use

Use when you have an unfiltered NanoSV SV call set from long-read (MinION/nanopore) data plus benchmark SV sites (1KG or Pacific Biosciences) and need to select a thresholded filter operating point trading precision against sensitivity.

## When NOT to use

- The SV call set is already filtered to a fixed operating point, so no threshold sweep is possible
- No benchmark SV catalog (1KG or Pacific Biosciences) is available to assign TP/FP labels to training calls
- The goal is SNV/indel variant calling from short-read Illumina data rather than long-read SV post-calling filtering

## Inputs

- Coordinate-sorted, indexed BAM of MinION nanopore reads mapped to a reference genome (e.g., LAST output converted via maf-convert and sambamba)
- NanoSV VCF call set for the NA12878 sample
- Benchmark SV sites from 1KG (1000 Genomes Project) and/or Pacific Biosciences call sets for TP/FP labeling
- Per-call feature table of 12 NanoSV-derived features (VAF, PID, CIPOS/CIEND, plength, MAPQ, local coverage, etc.)

## Outputs

- Labeled training table (each SV call tagged true positive / false positive via 1KG or PB benchmark overlap)
- Trained scikit-learn random forest classifier on the 12 SV features
- Precision-vs-sensitivity curve across score thresholds (cf. Supplementary Fig. 19)
- Selected optimal operating point reported as precision and sensitivity (~82% precision / ~75% sensitivity on training data)

## How to apply

Run NanoSV on the LAST-mapped, coordinate-sorted and indexed NA12878 nanopore BAM to obtain the raw SV call set. Label each call a true positive if its breakpoint overlaps a 1KG (1000 Genomes Project) or Pacific Biosciences benchmark SV site, a false positive otherwise. From the NanoSV output extract 12 features per call, including variant allele fraction (VAF), percentage of identity (PID), CIPOS/CIEND confidence intervals, pattern length (plength), MAPQ, and local coverage. Train a scikit-learn random forest classifier on the labeled set, then evaluate precision and sensitivity across score thresholds on the training data to locate the optimal operating point. The validation criterion is reproducing the paper's operating point of extremely stringent filtering that yields ~82% precision with ~75% estimated sensitivity (Supplementary Fig. 19).

## Related tools

- **NanoSV** (Generates the raw SV call set and per-call fields (VAF, PID, MAPQ, coverage) from which the 12 training features are extracted) — https://github.com/mroosmalen/nanosv
- **LAST** (Maps the nanopore reads to the reference; the README notes LAST alignments give the most accurate results for SV calling with NanoSV) — http://last.cbrc.jp/
- **sambamba** (Converts and sorts the LAST MAF-derived SAM into the sorted BAM required as NanoSV input) — https://github.com/biod/sambamba
- **scikit-learn** (Random forest classifier trained on the 12 labeled NanoSV features; threshold sweep over predicted scores locates the operating point)

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier(); rf.fit(X_train_12feat, y_tp_fp); scores = rf.predict_proba(X_train_12feat)[:,1]
```

## Evaluation signals

- The precision/sensitivity curve over score thresholds contains an operating point near 82% precision and 75% sensitivity on the training data, matching Supplementary Fig. 19
- Every training call carries a TP/FP label derived from overlap with a 1KG or Pacific Biosciences benchmark SV site
- The per-call feature vector contains exactly 12 features including VAF, PID, CIPOS/CIEND, plength, MAPQ, and local coverage extracted from NanoSV output
- The threshold sweep spans the full random forest score range and precision rises monotonically toward the stringent-filtering end
- The input NanoSV BAM is coordinate-sorted and indexed, and SV calls were made with LAST-based alignments (reported to give the most accurate NanoSV results)

## Limitations

- The 82%/75% metrics are reported on the training data (Supplementary Fig. 19), so they reflect the optimal operating point rather than an independent validation
- The ~75% estimated sensitivity is constrained by the paper's medium coverage (11–16x), intrinsic nanopore sequencing biases, and the deliberate choice of extremely stringent filtering
- Benchmark labeling depends on 1KG/PB site catalogs, so novel variants absent from these references are necessarily labeled false positives

## Evidence

- [supplementary] An optimal random forest model was selected with 82% precision and 75% sensitivity on the training data (Supplementary Fig. 19).: [evidence span withheld — anchor status: not_found]
- [discussion] we employ extremely stringent filtering that results in a good estimated sensitivity (~75%): "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [readme] NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data: "NanoSV is a software package that can be used to identify structural genomic variations in long-read sequencing data, such as data produced by Oxford Nanopore Technologies’ MinION"
- [readme] This BAM file needs to be coordinate-sorted and indexed.: "This BAM file needs to be coordinate-sorted and indexed."
