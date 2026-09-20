---
name: random-forest-classifier-training
description: Use when you have a raw NanoSV (or similar long-read) SV call set with
  per-call features and an independent benchmark truth set (e.g. 1KG or PB SV sites)
  and need a principled, stringent post-calling filter instead of hand-tuned cutoffs.
when_to_use_negative:
- You only need a discovery-grade call set without a precision benchmark — the paper's
  pipeline deliberately uses extremely stringent filtering that costs sensitivity
  (~75%), which is wrong for maximizing variant recovery
- No labeled benchmark truth set (1KG or PB SV sites) exists for your genome/sample,
  since the classifier is supervised and requires TP/FP labels
- Input reads are short reads (e.g. Illumina); NanoSV is designed for long reads from
  MinION/GridION/PromethION or PacBio RSII/Sequel instruments
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Produces the split/gapped-read-based SV call set (VCF) whose per-call features
    feed the random forest
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Read mapper used before SV calling; its alignments give the most accurate
    NanoSV results and thus the call features the classifier learns from
  repo: http://last.cbrc.jp/
- name: sambamba
  role: Converts and sorts LAST output (MAF→SAM→BAM) into the coordinate-sorted BAM
    required by NanoSV
  repo: https://github.com/biod/sambamba
- name: scikit-learn
  role: Implements the random forest classifier trained on the labeled SV feature
    table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:210ac811692759051c61ba5c6a5ec14af783d38961f50413c2de910c730e5a8d
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# random-forest-classifier-training

## Summary

Train a supervised random forest classifier (scikit-learn) on labeled NanoSV structural-variant calls to replace ad-hoc post-calling filtering, raising precision far above the ~54% baseline while retaining ~75% sensitivity. Calls are labeled true/false positives against 1KG and Pacific Biosciences benchmark SV sites and described by 12 NanoSV-derived features (VAF, PID, CIPOS/CIEND, plength, MAPQ, local coverage, etc.).

## When to use

Use when you have a raw NanoSV (or similar long-read) SV call set with per-call features and an independent benchmark truth set (e.g. 1KG or PB SV sites) and need a principled, stringent post-calling filter instead of hand-tuned cutoffs.

## When NOT to use

- You only need a discovery-grade call set without a precision benchmark — the paper's pipeline deliberately uses extremely stringent filtering that costs sensitivity (~75%), which is wrong for maximizing variant recovery
- No labeled benchmark truth set (1KG or PB SV sites) exists for your genome/sample, since the classifier is supervised and requires TP/FP labels
- Input reads are short reads (e.g. Illumina); NanoSV is designed for long reads from MinION/GridION/PromethION or PacBio RSII/Sequel instruments

## Inputs

- NanoSV SV call set (VCF) from mapped nanopore reads (e.g. LAST- or minimap2-aligned MinION BAM)
- Benchmark SV truth set (1KG / 1000 Genomes Project sites, Pacific Biosciences SV sites) for TP/FP labeling
- Per-call NanoSV feature table with 12 fields including VAF, PID, CIPOS, CIEND, plength, MAPQ, local coverage

## Outputs

- Trained random forest classifier (scikit-learn model object)
- Precision/sensitivity curve over score thresholds on the training data
- Selected high-precision operating point (~82% precision, ~75% sensitivity in the reference run) for filtering the SV call set

## How to apply

First run NanoSV on mapped long reads (LAST alignments gave the most accurate NanoSV results) to produce the SV call VCF, then split calls into true positives overlapping 1KG or Pacific Biosciences benchmark SV sites and false positives otherwise. Extract the 12 per-call features emitted by NanoSV — including variant allele fraction (VAF), percentage of identity (PID), confidence intervals CIPOS/CIEND, pattern length (plength), mapping quality (MAPQ) and local coverage. Train a scikit-learn random forest on this labeled feature matrix, then sweep the classifier score threshold across the training set and select an extremely stringent operating point that trades some sensitivity for high precision. The paper's target regime is >95% precision with ~75% estimated sensitivity (Supplementary Fig. 19 reports 82% precision / 75% sensitivity for the selected model), so verify your chosen threshold reproduces that balance rather than maximizing sensitivity.

## Related tools

- **NanoSV** (Produces the split/gapped-read-based SV call set (VCF) whose per-call features feed the random forest) — https://github.com/mroosmalen/nanosv
- **LAST** (Read mapper used before SV calling; its alignments give the most accurate NanoSV results and thus the call features the classifier learns from) — http://last.cbrc.jp/
- **sambamba** (Converts and sorts LAST output (MAF→SAM→BAM) into the coordinate-sorted BAM required by NanoSV) — https://github.com/biod/sambamba
- **scikit-learn** (Implements the random forest classifier trained on the labeled SV feature table)

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier(n_estimators=500); rf.fit(X12, y_tp_fp)
```

## Evaluation signals

- Classifier operating point reaches >95% precision on labeled calls; the paper's selected model reports 82% precision / 75% sensitivity on training data (Supplementary Fig. 19)
- Estimated sensitivity at the chosen threshold stays near ~75%, matching the paper's 'extremely stringent filtering' regime
- Fraction of FP labels correctly resolved: TPs should overlap 1KG or PB benchmark SV sites while retained FPs reflect the conservative calling steps that lower the novel-variant fraction
- Feature table completeness: all 12 features (VAF, PID, CIPOS/CIEND, plength, MAPQ, local coverage, etc.) extracted for every call before training
- Model generalization check: baseline post-calling precision (~54%) should be clearly exceeded after random forest filtering

## Limitations

- Stringent filtering reduces estimated sensitivity to ~75%; conservative SV calling and post-calling filtering also lower the fraction of novel variants relative to PB data
- Training is benchmark-dependent: labels rely on overlap with 1KG/PB truth sites, so classifier quality is capped by truth-set completeness
- Sensitivity is further limited by medium nanopore coverage (~11–16x) and intrinsic nanopore sequencing biases

## Evidence

- [discussion] stringent filtering with target sensitivity: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] conservative calling lowers novel fraction: "this is partly due to our conservative SV calling and post-calling filtering steps"
- [readme] NanoSV call basis: "NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments"
- [readme] mapper choice rationale: "We found that LAST alignments give the most accurate results for SV calling with NanoSV"
