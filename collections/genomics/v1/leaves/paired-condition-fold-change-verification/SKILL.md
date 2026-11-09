---
name: paired-condition-fold-change-verification
description: Use when matched dZ-DNA and native control BAMs with kinetic tags (fi,
  ri, fp, rp) are available and you must quantify/verify Z-vs-A fold changes in pulse
  width (PW) or IPD for a specific sequence motif.
when_to_use_negative:
- You need to detect dZ-DNA presence itself (use Z-Calling's read-level SVM/MLP classification
  instead of fold-change verification)
- Context k-mers contain multiple adjacent A/Z bases — the analysis restricts to k-mers
  with a single A/Z target to avoid interference
- Data lacks preserved kinetics (CCS generated without --keep-kinetics) or has < 3
  forward/reverse passes — fold-change ratios are then not comparable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Z-Calling
  role: filter_bam.py applies the kinetic-tag, PW/IPD-ratio and pass filters; z-bam2txt
    -zp converts the aligned BAM into the per-base kinetics table used to compute
    fold changes
  repo: https://github.com/xiaochuanle/Z-Calling
- name: pbmm2
  role: aligns filtered CCS reads to the reference (--preset CCS --sort) so PW/IPD
    values can be binned by 21-mer sequence context
- name: Python (pandas/numpy)
  role: aggregates forward-strand PW/IPD per motif family and computes mean ± SD Z/A
    fold-change ratios
- name: samtools
  role: optional conversion of filtered BAM to FASTA for downstream mapping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:2ae9b910cd1aa9c5dc262d26ec1fe924cc96e2cd32b88e8e27a5f981524d0e3e
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
  - Python
  - Python (pandas/numpy)
  - Z-Calling
  - minimap2
  - pbmm2
  - samtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# paired-condition fold-change verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies context-dependent polymerase kinetic shifts caused by dA/dZ (2,6-diaminopurine) substitution by computing mean ± SD PW/IPD fold-change ratios for matched motif families (e.g., TTZTT vs TTATT, TTZGG vs TTAGG) from filtered PacBio HiFi kinetics and confirming they reproduce reported values (1.46 ± 0.12 and 0.84 ± 0.08).

## When to use

Use when matched dZ-DNA and native control BAMs with kinetic tags (fi, ri, fp, rp) are available and you must quantify/verify Z-vs-A fold changes in pulse width (PW) or IPD for a specific sequence motif.

## When NOT to use

- You need to detect dZ-DNA presence itself (use Z-Calling's read-level SVM/MLP classification instead of fold-change verification)
- Context k-mers contain multiple adjacent A/Z bases — the analysis restricts to k-mers with a single A/Z target to avoid interference
- Data lacks preserved kinetics (CCS generated without --keep-kinetics) or has < 3 forward/reverse passes — fold-change ratios are then not comparable

## Inputs

- Raw dZ-DNA and native (control) PacBio CCS BAM files carrying kinetic tags (fi, ri, fp, rp)
- Reference FASTA of the exact genome/amplicon used for alignment (e.g., E. coli reference)
- Pre-trained Z-Calling models (optional, if Z probabilities per base are needed alongside kinetics)

## Outputs

- Filtered/aligned BAM (pbmm2 --preset CCS --sort)
- Per-base kinetics table from z-bam2txt -zp (QueryName, ReadID, Strand, QueryPos, Base, RefName, RefPos, Probability)
- motif_kinetic_ratios.tsv with mean ± SD Z/A fold-change ratios for PW and IPD per motif family

## How to apply

First filter raw dZ-DNA and native BAMs with Z-Calling's filter_bam.py so only reads with kinetic tags (fi, ri, fp, rp), median PW/median IPD ratio >= 0.3 and below the 99th percentile, and fn >= 3 and rn >= 3 passes survive. Align with pbmm2 (--preset CCS --sort), then run z-bam2txt -zp supplying the exact reference used for alignment to obtain a per-base kinetics table. Reconstruct 21-mer contexts centered on each A/Z target, keeping only single-A/Z contexts with >= 10 observations in both conditions (to avoid interference among adjacent A/Zs), and aggregate forward-strand PW and IPD per motif family. Compute mean ± SD Z/A fold-change ratios per motif with pandas/numpy (e.g., TTZTT vs TTATT and TTZGG vs TTAGG), and treat the analysis as verified only if the ratios reproduce the reported values (PW 1.46 ± 0.12 for TTZTT, p = 2.75e-306; 0.84 ± 0.08 for TTZGG, p = 3.95e-48) within the reported 0.5–1.5-fold Z-induced shift range.

## Related tools

- **Z-Calling** (filter_bam.py applies the kinetic-tag, PW/IPD-ratio and pass filters; z-bam2txt -zp converts the aligned BAM into the per-base kinetics table used to compute fold changes) — https://github.com/xiaochuanle/Z-Calling
- **pbmm2** (aligns filtered CCS reads to the reference (--preset CCS --sort) so PW/IPD values can be binned by 21-mer sequence context)
- **Python (pandas/numpy)** (aggregates forward-strand PW/IPD per motif family and computes mean ± SD Z/A fold-change ratios)
- **samtools** (optional conversion of filtered BAM to FASTA for downstream mapping)

## Examples

```
pbmm2 align REF.fasta FilteredBAM FilteredMappingBAM.bam --preset CCS --sort && build/z-bam2txt -zp -t 16 FilteredMappingBAM.bam - REF.fasta > kinetics.tsv
```

## Evaluation signals

- Computed TTZTT vs TTATT PW fold change reproduces 1.46 ± 0.12 (p = 2.75e-306)
- Computed TTZGG vs TTAGG PW fold change reproduces 0.84 ± 0.08 (p = 3.95e-48)
- All ratios fall within the reported 0.5–1.5-fold Z-induced kinetic shift range
- Every analyzed 21-mer has exactly one A/Z target and >= 10 observations in both dZ and native conditions
- Reads passing filters satisfy median PW/median IPD >= 0.3 (below the 99th percentile), fn >= 3 and rn >= 3

## Limitations

- The subtle Z-induced shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance in PacBio kinetic measurements, limiting hybrid-sample quantification
- Z effects are context-dependent: PW can be elongated in one motif (TTZTT) and shortened in another (TTZGG), so single-motif results do not generalize
- SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures

## Evidence
