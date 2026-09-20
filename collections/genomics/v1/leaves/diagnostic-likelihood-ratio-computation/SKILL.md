---
name: diagnostic-likelihood-ratio-computation
description: Use when evaluating a binary read-level classifier (e.g., the Z-Calling
  SVM k21ReadClassifier) on datasets with known ground-truth labels (dZ-DNA vs canonical
  DNA) and you need TPR, FPR, and LR+ as performance/diagnostic metrics.
when_to_use_negative:
- Sample contains hybrid or diluted A/Z reads rather than fully substituted dZ-genomes
  — the SVM read classifier is optimized for fully substituted dZ-genomes and misses
  hybrid reads with diluted kinetic signatures, so LR+ read-level classification will
  under-detect.
- Goal is single-nucleotide Z/A discrimination within reads rather than molecule-level
  dZ-DNA vs canonical classification — use the k11-mixed-AZ base-calling module (AUC
  0.942–0.952) instead of the read-level SVM.
- Reads are shorter than 500 bp or contain fewer than 100 A+Z bases — they are excluded
  from classification and will appear unclassified, biasing the confusion counts.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Z-Calling
  role: Provides the SVM read classifier (build/z-calling-read predict with model/svm/k21ReadClassifier,
    min_len=500, intervals=100) whose '+'/'-' predictions are tabulated into confusion
    counts for TPR/FPR/LR+ computation, plus py/SVM_LR_analysis.py for downstream
    LR analysis
  repo: https://github.com/xiaochuanle/Z-Calling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:f202123b81ec3e184ba8fe317a249c5213eb65b741fd40f4542bf7fd9e03e7da
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - Z-Calling
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# diagnostic-likelihood-ratio-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes diagnostic performance metrics (TPR, FPR, and positive likelihood ratio LR+ = TPR/FPR) for the Z-Calling SVM read classifier by tabulating per-dataset confusion counts of '+'/'-' read predictions against known dZ-positive and canonical DNA dataset labels; benchmarking requires a minimum LR+ of at least 490, consistent with zero false positive discovery.

## When to use

Use when evaluating a binary read-level classifier (e.g., the Z-Calling SVM k21ReadClassifier) on datasets with known ground-truth labels (dZ-DNA vs canonical DNA) and you need TPR, FPR, and LR+ as performance/diagnostic metrics.

## When NOT to use

- Sample contains hybrid or diluted A/Z reads rather than fully substituted dZ-genomes — the SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures, so LR+ read-level classification will under-detect.
- Goal is single-nucleotide Z/A discrimination within reads rather than molecule-level dZ-DNA vs canonical classification — use the k11-mixed-AZ base-calling module (AUC 0.942–0.952) instead of the read-level SVM.
- Reads are shorter than 500 bp or contain fewer than 100 A+Z bases — they are excluded from classification and will appear unclassified, biasing the confusion counts.

## Inputs

- Raw PacBio CCS BAM with kinetic tags (fi, ri, fp, rp)
- Filtered BAM from py/filter_bam.py (kinetic tags present, Median_PW/Median_IPD >= 0.3 and <= 99th percentile, fn >= 3, rn >= 3)
- ZP-tagged output BAM from build/z-calling-base -k 21 with model/k21-full-ZA/scripted_m21.pth
- Pre-trained SVM model model/svm/k21ReadClassifier
- Known dataset labels: four dZ-positive datasets (amplicons plus SH-Ab 15497 phage) and seven canonical DNA datasets

## Outputs

- Tab-separated SVM result file pairing each read name with '+' (dZ-DNA), '-' (canonical), or unclassified
- Per-dataset confusion counts (TP, FP, TN, FN) against known labels
- TPR, FPR, and positive likelihood ratio LR+ = TPR/FPR per dataset
- SVM.LR.tsv from py/SVM_LR_analysis.py when used for taxonomic source LR analysis

## How to apply

First filter the input PacBio CCS BAM with py/filter_bam.py (keep reads carrying kinetic tags fi, ri, fp, rp; discard reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile; require at least 3 passes on both strands, fn >= 3 and rn >= 3). Run build/z-calling-base with -k 21 using model/k21-full-ZA/scripted_m21.pth to generate a ZP-tagged BAM, then run build/z-calling-read predict with model/svm/k21ReadClassifier, min_len=500 and intervals=100, to produce a TSV pairing each read name with '+' (dZ-DNA), '-' (canonical), or unclassified. Tabulate per-dataset confusion counts against the known dataset labels (four dZ-positive datasets — amplicons plus the Acinetobacter phage SH-Ab 15497 — and seven canonical DNA datasets), compute TPR = TP/(TP+FN), FPR = FP/(FP+TN), and LR+ = TPR/FPR. Validate that per-dataset TPR falls within 93.44–99.72%, FPR within 0.02–0.19%, and that the minimum LR+ across datasets is at least 490; for mixed-dataset taxonomic source attribution, feed the SVM result into py/SVM_LR_analysis.py to produce SVM.LR.tsv.

## Related tools

- **Z-Calling** (Provides the SVM read classifier (build/z-calling-read predict with model/svm/k21ReadClassifier, min_len=500, intervals=100) whose '+'/'-' predictions are tabulated into confusion counts for TPR/FPR/LR+ computation, plus py/SVM_LR_analysis.py for downstream LR analysis) — https://github.com/xiaochuanle/Z-Calling

## Examples

```
python py/SVM_LR_analysis.py --SVM z-calling-read-Result.tsv --read_ref read_RefContig.tsv --output SVM.LR.tsv
```

## Evaluation signals

- Per-dataset TPR falls within 93.44–99.72% across the four dZ-DNA datasets
- Per-dataset FPR falls within 0.02–0.19% across the seven canonical DNA datasets
- Minimum LR+ across datasets is at least 490, consistent with zero false positive discovery in benchmarking
- Arithmetic invariant check: each reported LR+ equals TPR divided by FPR for the same dataset
- SVM output TSV contains only read names paired with '+', '-', or an unclassified marker; unclassified reads should correspond to failed length criteria (min_len >= 500)

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures; it enforces a high confidence threshold to minimize false positives, limiting hybrid-sample quantification
- Inherent noise of PacBio kinetic measurements means the subtle Z-induced shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance
- Reads shorter than 500 bp, or not divisible for exactly 100 intervals of feature extraction, are left unclassified and must be excluded from TPR/FPR denominators

## Evidence

- [results] The Z-Calling SVM read classifier achieved true positive rates of 93.44–99.72% across the four dZ-DNA datasets: [evidence span withheld — anchor status: not_found]
- [discussion] stems primarily from the model's optimization for fully substituted dZ-genomes. To robustly identify organisms like phages while minimizing false positives, the SVM enforces a high confidence: [evidence span withheld — anchor status: not_found]
