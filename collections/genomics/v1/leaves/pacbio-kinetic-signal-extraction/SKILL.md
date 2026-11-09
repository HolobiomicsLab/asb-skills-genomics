---
name: pacbio-kinetic-signal-extraction
description: Use when PacBio CCS BAM files contain kinetic tags (fi, ri, fp, rp) and
  the goal is per-base PW/IPD feature extraction around A/Z target bases for dZ-DNA
  detection or kinetic shift analysis.
when_to_use_negative:
- Input reads lack kinetic tags (fi, ri, fp, rp) or CCS BAMs were produced without
  '--keep-kinetics', since PW/IPD signals are absent
- The goal is ONT ionic-current analysis rather than PacBio polymerase kinetics —
  the article notes ONT relies on ionic-current perturbations, a different signal
  modality
- You only need read-level dZ-DNA vs canonical classification — the SVM read classifier
  (z-calling-read, min_len >= 500, intervals = 100) is the appropriate tool instead
  of per-base k-mer extraction
tools:
- name: Z-Calling
  role: Provides filter_bam.py for kinetic-quality read filtering and z-bam2txt for
    converting ZP-tagged BAMs into per-base PW/IPD probability tables
  repo: https://github.com/xiaochuanle/Z-Calling
- name: pbmm2
  role: Aligns the filtered CCS reads to the reference genome (--preset CCS --sort)
    so per-base kinetic values can be mapped to reference coordinates
- name: Python (pandas/numpy)
  role: Aggregates forward-strand PW and IPD per motif family, computes mean ± SD
    Z/A fold-change ratios, and writes motif_kinetic_ratios.tsv
- name: samtools
  role: Converts filtered BAM to FASTA (samtools fasta -@8) when downstream minimap2
    mapping of Z-calling reads is needed
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:5acac51c9cdf17f07fb76f1ddd8abd3aa1a2b6f0e517e135f6c74ca6a26bf3ca
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
  - PacBio CCS
  - PacBio Circular Consensus Sequencing (CCS)
  - PyTorch
  - Python
  - Python (pandas/numpy)
  - Z-Calling
  - minimap2
  - pbmm2
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# pacbio-kinetic-signal-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract pulse width (PW) and inter-pulse duration (IPD) kinetic signals from PacBio HiFi/CCS BAMs, filter reads by kinetic quality (fi/ri/fp/rp tags, Median_PW/Median_IPD ≥ 0.3 and ≤ 99th percentile, fn/rn ≥ 3), and build single-A/Z 21-mer context k-mer feature tables for Z/A discrimination and downstream modeling.

## When to use

Use when PacBio CCS BAM files contain kinetic tags (fi, ri, fp, rp) and the goal is per-base PW/IPD feature extraction around A/Z target bases for dZ-DNA detection or kinetic shift analysis.

## When NOT to use

- Input reads lack kinetic tags (fi, ri, fp, rp) or CCS BAMs were produced without '--keep-kinetics', since PW/IPD signals are absent
- The goal is ONT ionic-current analysis rather than PacBio polymerase kinetics — the article notes ONT relies on ionic-current perturbations, a different signal modality
- You only need read-level dZ-DNA vs canonical classification — the SVM read classifier (z-calling-read, min_len >= 500, intervals = 100) is the appropriate tool instead of per-base k-mer extraction

## Inputs

- PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp) from dZ-DNA and canonical DNA samples (e.g., E. coli BL21, Drosophila melanogaster S2)
- Reference FASTA used for alignment (e.g., E. coli genome via pbmm2)
- Optional: pretrained model path (model/k21-full-ZA/scripted_m21.pth or model/k11-mixed-AZ/scripted_m11.pth) when Z probabilities must be computed before table extraction

## Outputs

- Filtered BAM retaining only reads passing kinetic quality filters
- Tab-delimited per-base table from z-bam2txt (QueryName, ReadID, Strand, QueryPos, Base, RefName, RefPos, Probability for mapped reads)
- 21-mer context k-mer feature table of PW/IPD values centered on A/Z target bases
- Motif-level Z/A fold-change summary (e.g., motif_kinetic_ratios.tsv: PW 1.46 ± 0.12 for TTZTT, 0.84 ± 0.08 for TTZGG)

## How to apply

Start with CCS BAMs generated with the '--keep-kinetics' option, then run Z-Calling's filter_bam.py keeping only reads with kinetic tags, Median_PW/Median_IPD ratio >= 0.3 and below the 99th percentile, and at least 3 passes on both strands (fn >= 3, rn >= 3). If needed, align filtered reads with pbmm2 (pbmm2 align REF.fasta FilteredBAM --preset CCS --sort). Convert the filtered BAM to a per-base probability table with z-bam2txt -zp -t 16, supplying the exact reference FASTA used for alignment so mapping coordinates remain consistent. Reconstruct context k-mers (21-mers for the full-substitution analysis, 11-mers for mixed A/Z models) centered on the target base, retaining only k-mers that contain exactly one A/Z base and at least 10 recorded observations in both dZ-DNA and canonical conditions to avoid interference among adjacent A/Z bases. Aggregate forward-strand PW and IPD values per motif family (e.g., TTZTT vs TTATT) and compute Z/A fold-change ratios; this feature matrix feeds RFC/MLP training or direct kinetic-shift comparison.

## Related tools

- **Z-Calling** (Provides filter_bam.py for kinetic-quality read filtering and z-bam2txt for converting ZP-tagged BAMs into per-base PW/IPD probability tables) — https://github.com/xiaochuanle/Z-Calling
- **pbmm2** (Aligns the filtered CCS reads to the reference genome (--preset CCS --sort) so per-base kinetic values can be mapped to reference coordinates)
- **Python (pandas/numpy)** (Aggregates forward-strand PW and IPD per motif family, computes mean ± SD Z/A fold-change ratios, and writes motif_kinetic_ratios.tsv)
- **samtools** (Converts filtered BAM to FASTA (samtools fasta -@8) when downstream minimap2 mapping of Z-calling reads is needed)

## Examples

```
python filter_bam.py -b raw.bam -o filtered.bam && build/z-bam2txt -zp -t 16 filtered.bam Ref.fasta > kinetics.tsv
```

## Evaluation signals

- Filtering reproducibility: retained reads must all have fn >= 3 and rn >= 3 and Median_PW / Median_IPD between 0.3 and the 99th percentile
- Every retained context k-mer contains exactly one A/Z target base and has at least 10 observations in both dZ-DNA and canonical conditions
- Computed PW Z/A fold changes match reported values: 1.46 ± 0.12 (p = 2.75e−306) for TTZTT and 0.84 ± 0.08 (p = 3.95e−48) for TTZGG, within the 0.5–1.5-fold shift range
- z-bam2txt output has 8 columns (mapped reads) or 6 columns (unmapped reads) as specified in the README, with ZP-derived probability values present
- Downstream RFC trained on the extracted PW/IPD features reaches the reported baseline accuracy of 0.842 ± 0.018 with PW features dominating the importance ranking

## Limitations

- The subtle Z-induced kinetic shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance due to inherent noise of PacBio kinetic measurements, limiting hybrid-sample quantification
- Reads shorter than 500 bp or with fewer than 100 A+Z bases are excluded due to the limited number of A/Z bases; datasets must meet a quality threshold of at least 60,000 reads with pass numbers >= 3
- Min_len for read-level classification must be >= 500 and the SVM intervals argument must be exactly 100, or reads remain unclassified
- The reference FASTA supplied to z-bam2txt must be the exact file used for alignment if zfreq analysis follows, otherwise mapping coordinates will not match

## Evidence
