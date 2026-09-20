---
name: svm-read-classification-inference
description: Use when input is a kinetics-preserving PacBio CCS BAM (--keep-kinetics
  fi/ri/fp/rp tags) and the goal is a per-read dZ-DNA vs canonical DNA label for molecule-level
  detection.
when_to_use_negative:
- Input is single-nucleotide site-level Z/A discrimination in hybrid A/Z sequences
  — use the k11-mixed-AZ MLP base-calling module (scripted_m11.pth, -k 11) instead
- BAM lacks kinetic tags (fi, ri, fp, rp) or was generated without --keep-kinetics;
  filtering and the kinetic-signature models cannot be applied
- Samples contain hybrid/diluted reads with mixed Z content — the SVM is optimized
  for fully substituted dZ-genomes and will miss diluted kinetic signatures
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: 'Provides the full read-classification pipeline: py/filter_bam.py read filtering,
    build/z-calling-base (-k 21, k21-full-ZA model) for ZP tagging, and build/z-calling-read
    predict (k21ReadClassifier SVM) for per-read dZ-DNA labels'
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Source data modality; CCS BAMs generated with --keep-kinetics supply the pulse
    width (PW) and inter-pulse duration (IPD) kinetic signals the classifier consumes
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:51130ac54a25a228aa3eb816d2bf26c3e59cafad58fa52a5f1feeeac0e321d46
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
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# svm-read-classification-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify PacBio HiFi/CCS reads as dZ-DNA (+) versus canonical DNA (-) using Z-Calling's k21 MLP base-caller plus the pre-trained k21ReadClassifier SVM, achieving TPR 93.44–99.72% and FPR 0.02–0.19% on benchmark datasets.

## When to use

Use when input is a kinetics-preserving PacBio CCS BAM (--keep-kinetics fi/ri/fp/rp tags) and the goal is a per-read dZ-DNA vs canonical DNA label for molecule-level detection.

## When NOT to use

- Input is single-nucleotide site-level Z/A discrimination in hybrid A/Z sequences — use the k11-mixed-AZ MLP base-calling module (scripted_m11.pth, -k 11) instead
- BAM lacks kinetic tags (fi, ri, fp, rp) or was generated without --keep-kinetics; filtering and the kinetic-signature models cannot be applied
- Samples contain hybrid/diluted reads with mixed Z content — the SVM is optimized for fully substituted dZ-genomes and will miss diluted kinetic signatures

## Inputs

- PacBio CCS BAM file with kinetic tags (fi, ri, fp, rp), from sequencing run with --keep-kinetics
- Filtered BAM produced by py/filter_bam.py (kinetic-tag-preserving, Median_PW/Median_IPD quality and fn>=3/rn>=3 pass filters applied)
- ZP-tagged BAM output of build/z-calling-base -k 21 with model/k21-full-ZA/scripted_m21.pth
- Pre-trained SVM model model/svm/k21ReadClassifier (shipped in the Z-Calling repository)

## Outputs

- Tab-separated results file with two columns: read name and classification (+ dZ-DNA, - canonical DNA, otherwise unclassified)
- Per-dataset confusion counts from which TPR, FPR, and LR+ = TPR/FPR can be computed against known dataset labels

## How to apply

First run py/filter_bam.py on the raw BAM to retain reads carrying kinetic tags (fi, ri, fp, rp), discarding reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile and requiring at least 3 passes on both strands (fn >= 3, rn >= 3). Then run build/z-calling-base with -k 21 on the filtered BAM using model/k21-full-ZA/scripted_m21.pth, which adds ZP tags recording per-base Z probabilities intended for full-read dZ-DNA detection. Finally run build/z-calling-read predict on the ZP-tagged BAM with model/svm/k21ReadClassifier, min_len=500 and intervals=100 (intervals must be exactly 100 for the current model), yielding a TSV of read names and '+' (dZ-DNA), '-' (canonical), or unclassified labels. Decision point: reads failing the length criterion are reported as unclassified rather than forced into a class, so tabulate classified reads only when computing accuracy metrics.

## Related tools

- **Z-Calling** (Provides the full read-classification pipeline: py/filter_bam.py read filtering, build/z-calling-base (-k 21, k21-full-ZA model) for ZP tagging, and build/z-calling-read predict (k21ReadClassifier SVM) for per-read dZ-DNA labels) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Source data modality; CCS BAMs generated with --keep-kinetics supply the pulse width (PW) and inter-pulse duration (IPD) kinetic signals the classifier consumes)

## Examples

```
build/z-calling-read predict ZP_tagged.bam results.tsv model/svm/k21ReadClassifier 500 100
```

## Evaluation signals

- Per-dataset true positive rates across the four dZ-DNA datasets (amplicons plus SH-Ab 15497 phage) fall within the reported 93.44–99.72% range
- False positive rates across the seven canonical DNA datasets (E. coli BL21, yeast S288C, fruit fly S2, HG002, HG00106, Arabidopsis, rice, zebrafish) fall within 0.02–0.19%
- Minimum positive likelihood ratio LR+ = TPR/FPR across datasets is at least 490, consistent with zero false positive discovery in benchmarking
- Output TSV contains exactly two columns per line and labels restricted to '+', '-', or unclassified; reads shorter than min_len 500 appear as unclassified
- Filtered BAM retains only reads with fi, ri, fp, rp tags, Median_PW/Median_IPD in [0.3, 99th percentile], and fn >= 3, rn >= 3

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures; the subtle Z-induced shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance
- min_len must be >= 500 and intervals must be exactly 100 for the current model; reads failing these criteria are emitted as unclassified
- The high-confidence SVM threshold enforces a low false positive rate at the cost of sensitivity, appropriate for identifying organisms like phages rather than quantifying dZ fraction in mixed samples

## Evidence

- [methods] Filter each input BAM with py/filter_bam.py, keeping reads carrying kinetic tags (fi, ri, fp, rp), discarding reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile: [evidence span withheld — anchor status: not_found]
