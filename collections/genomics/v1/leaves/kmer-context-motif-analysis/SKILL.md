---
name: kmer-context-motif-analysis
description: Use when PacBio HiFi/CCS BAMs with kinetic tags (fi, ri, fp, rp) are
  available and the goal is to test whether a modified base (e.g., dZ vs dA) shifts
  PW or IPD in a sequence-context-dependent manner across specific motif families
  (TTZTT vs TTATT, TTZGG vs TTAGG).
when_to_use_negative:
- The input BAMs lack kinetic tags (fi, ri, fp, rp), so PW/IPD signals cannot be extracted
- The goal is single-nucleotide Z/A base calling or dZ-DNA read classification rather
  than kinetic motif analysis — use z-calling-base (k11-mixed-AZ or k21-full-ZA models)
  instead
- Candidate k-mers contain multiple adjacent A/Z bases, which the analysis explicitly
  excludes to avoid interference among neighboring A/Zs
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling (filter_bam.py)
  role: Filters raw dZ-DNA/native BAMs for kinetic tags, median PW/median IPD >= 0.3
    and < 99th percentile, and fn/rn >= 3 passes before context extraction
  repo: https://github.com/xiaochuanle/Z-Calling
- name: Z-Calling (z-bam2txt)
  role: Converts the filtered mapping BAM to a per-base probability/context table
    from which 21-mer contexts around A/Z bases are reconstructed
  repo: https://github.com/xiaochuanle/Z-Calling
- name: pbmm2
  role: Aligns filtered CCS reads to the reference (pbmm2 align REF.fasta FilteredBAM
    --preset CCS --sort) prior to per-base table conversion
- name: Python (pandas/numpy)
  role: Aggregates forward-strand PW/IPD per 21-mer motif family, computes mean ±
    SD Z/A fold-change ratios, and writes motif_kinetic_ratios.tsv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:58dc5d52613f1f16bd6954eaaf29b41c03e0a1d38edfcfa4dd17fb4106641946
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
  - Z-Calling (filter_bam.py)
  - Z-Calling (z-bam2txt)
  - minimap2
  - pbmm2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# kmer-context-motif-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs sequence-context k-mers (e.g., 21-mers) centered on A/Z target bases from PacBio CCS kinetic tables and computes per-motif Z/A fold-change ratios of polymerase pulse width (PW) and inter-pulse duration (IPD), revealing context-dependent kinetic signatures of the 2,6-diaminopurine (Z) base.

## When to use

Use when PacBio HiFi/CCS BAMs with kinetic tags (fi, ri, fp, rp) are available and the goal is to test whether a modified base (e.g., dZ vs dA) shifts PW or IPD in a sequence-context-dependent manner across specific motif families (TTZTT vs TTATT, TTZGG vs TTAGG).

## When NOT to use

- The input BAMs lack kinetic tags (fi, ri, fp, rp), so PW/IPD signals cannot be extracted
- The goal is single-nucleotide Z/A base calling or dZ-DNA read classification rather than kinetic motif analysis — use z-calling-base (k11-mixed-AZ or k21-full-ZA models) instead
- Candidate k-mers contain multiple adjacent A/Z bases, which the analysis explicitly excludes to avoid interference among neighboring A/Zs

## Inputs

- Raw PacBio CCS BAM files of dZ-DNA and native (unmodified) DNA containing kinetic tags (fi, ri, fp, rp)
- Reference genome FASTA used for alignment (must be the exact file used with pbmm2)
- Aligned/filtered mapping BAM with ZP probability tags

## Outputs

- Per-base probability/context table from z-bam2txt (tab-delimited: QueryName, ReadID, Strand, QueryPos, Base, RefName, RefPos, Probability)
- Filtered set of 21-mer contexts centered on single A/Z targets with >=10 observations per condition
- motif_kinetic_ratios.tsv with mean ± SD Z/A fold-change ratios for PW and IPD per motif family

## How to apply

First filter raw dZ-DNA and native BAMs with Z-Calling's filter_bam.py, keeping reads carrying kinetic tags (fi, ri, fp, rp) with median PW / median IPD ratio >= 0.3 and below the 99th percentile, and with >= 3 passes on both forward (fn >= 3) and reverse (rn >= 3) strands. Align the filtered reads to the reference (e.g., E. coli) with pbmm2 align --preset CCS --sort, then convert the mapped BAM to a per-base probability table with z-bam2txt -zp, supplying the exact reference FASTA used for alignment so mapping coordinates match. From the table, reconstruct 21-mer contexts centered on each A/Z base and retain only contexts containing a single A/Z target (to avoid interference among adjacent A/Zs) with at least 10 observations in each condition. Aggregate forward-strand PW and IPD values per motif family and compute mean ± SD Z/A fold-change ratios with Python (pandas/numpy), writing a motif_kinetic_ratios.tsv; apply statistical testing (e.g., t-tests with BH correction) to assess significance.

## Related tools

- **Z-Calling (filter_bam.py)** (Filters raw dZ-DNA/native BAMs for kinetic tags, median PW/median IPD >= 0.3 and < 99th percentile, and fn/rn >= 3 passes before context extraction) — https://github.com/xiaochuanle/Z-Calling
- **Z-Calling (z-bam2txt)** (Converts the filtered mapping BAM to a per-base probability/context table from which 21-mer contexts around A/Z bases are reconstructed) — https://github.com/xiaochuanle/Z-Calling
- **pbmm2** (Aligns filtered CCS reads to the reference (pbmm2 align REF.fasta FilteredBAM --preset CCS --sort) prior to per-base table conversion)
- **Python (pandas/numpy)** (Aggregates forward-strand PW/IPD per 21-mer motif family, computes mean ± SD Z/A fold-change ratios, and writes motif_kinetic_ratios.tsv)

## Examples

```
build/z-bam2txt -zp -t 16 FilteredMappingBAM.bam Ref.fasta > Result.tsv
```

## Evaluation signals

- Computed PW Z/A fold-change reproduces the reported values: 1.46 ± 0.12 (p = 2.75e−306) for TTZTT vs TTATT and 0.84 ± 0.08 (p = 3.95e−48) for TTZGG vs TTAGG at base position 12
- Z-induced kinetic shift ratios fall within the reported 0.5–1.5-fold range
- Every retained 21-mer context contains exactly one A/Z target and has >= 10 observations in both conditions
- Only reads passing median PW / median IPD >= 0.3, < 99th percentile, and fn/rn >= 3 passes enter the analysis
- motif_kinetic_ratios.tsv is produced with per-motif PW and IPD mean ± SD ratios

## Limitations

- The subtle Z-induced kinetic shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance due to inherent PacBio kinetic measurement noise, limiting quantification in hybrid samples
- Only k-mers with a single A/Z target are analyzable; adjacent A/Z bases interfere and are excluded
- Contexts must have at least 10 recorded observations in both conditions, so rare motif families are dropped

## Evidence

- [results] context-dependent PW findings: [evidence span withheld — anchor status: not_found]
