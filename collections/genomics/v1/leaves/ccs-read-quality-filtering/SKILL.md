---
name: ccs-read-quality-filtering
description: Use when raw dZ-DNA or native PacBio HiFi/CCS BAMs must be cleaned before
  kinetic-signal analysis (PW/IPD) or Z-Calling, i.e. when reads may lack fi/ri/fp/rp
  tags, have aberrant PW/IPD ratios, or too few polymerase passes.
when_to_use_negative:
- Input BAMs already carry kinetic tags and pass QC (e.g. the paper's own benchmark
  CCS outputs generated with '--keep-kinetics' that were pre-filtered) — re-filtering
  can bias pass-count distributions
- Analysis targets ONT ionic-current data or HPLC nucleotide-composition measurements
  rather than PacBio pulse kinetics — the PW/IPD thresholds do not apply
- Reads are intended for taxonomic identification only via sequence divergence (minimap2
  dv tag) where kinetics are not required
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: Supplies filter_bam.py, the script that enforces the kinetic-tag, Median_PW/Median_IPD,
    and per-strand pass-count filters
  repo: https://github.com/xiaochuanle/Z-Calling
- name: pbmm2
  role: Optionally aligns the filtered BAM to the reference with --preset CCS --sort
    when the raw input was unmapped
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Generates the source BAMs; must be run with '--keep-kinetics' to preserve
    PW and IPD tags needed for filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:0991f230ab5474122845cbb08de77c26db30b37d7a149aa40ad663da38023fbb
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
  - Python
  - Z-Calling
  - minimap2
  - pbmm2
  - samtools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# ccs-read-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-processes PacBio CCS BAM files by retaining only reads carrying kinetic tags (fi, ri, fp, rp), enforcing signal-quality (median PW / median IPD >= 0.3 and below the 99th percentile), and requiring >= 3 passes per strand (fn >= 3, rn >= 3), so downstream kinetic and Z-calling analyses operate on reliable pulse data.

## When to use

Use when raw dZ-DNA or native PacBio HiFi/CCS BAMs must be cleaned before kinetic-signal analysis (PW/IPD) or Z-Calling, i.e. when reads may lack fi/ri/fp/rp tags, have aberrant PW/IPD ratios, or too few polymerase passes.

## When NOT to use

- Input BAMs already carry kinetic tags and pass QC (e.g. the paper's own benchmark CCS outputs generated with '--keep-kinetics' that were pre-filtered) — re-filtering can bias pass-count distributions
- Analysis targets ONT ionic-current data or HPLC nucleotide-composition measurements rather than PacBio pulse kinetics — the PW/IPD thresholds do not apply
- Reads are intended for taxonomic identification only via sequence divergence (minimap2 dv tag) where kinetics are not required

## Inputs

- Raw PacBio CCS BAM with kinetic tags (fi, ri, fp, rp), mapped or unmapped
- Reference genome FASTA (e.g. E. coli) if alignment of filtered reads is needed

## Outputs

- Filtered BAM retaining only reads with valid kinetic tags, pass counts, and PW/IPD quality (FilteredBAM)
- Optionally, a sorted aligned FilteredMappingBAM.bam from pbmm2 --preset CCS

## How to apply

Run Z-Calling's filter_bam.py on the raw (mapped or unmapped) BAM with kinetic tags present, discarding reads whose Median_PW / Median_IPD falls below 0.3 or above the 99th percentile, and keeping only reads with at least 3 forward (fn >= 3) and 3 reverse (rn >= 3) passes. Additional quality gates from the paper include >= 60,000 reads per dataset and read length >= 500 bp with >= 100 A+Z bases for classification contexts. If the input was unmapped, align the filtered BAM to the reference with pbmm2 (pbmm2 align REF.fasta FilteredBAM --preset CCS --sort). Rationale: reads lacking kinetics or with distorted PW/IPD ratios corrupt pulse-width and inter-pulse-duration measurements that Z-Calling and downstream 21-mer kinetic analyses depend on.

## Related tools

- **Z-Calling** (Supplies filter_bam.py, the script that enforces the kinetic-tag, Median_PW/Median_IPD, and per-strand pass-count filters) — https://github.com/xiaochuanle/Z-Calling
- **pbmm2** (Optionally aligns the filtered BAM to the reference with --preset CCS --sort when the raw input was unmapped)
- **PacBio Circular Consensus Sequencing (CCS)** (Generates the source BAMs; must be run with '--keep-kinetics' to preserve PW and IPD tags needed for filtering)

## Examples

```
python py/filter_bam.py -b RawBAM.bam -o FilteredBAM.bam && pbmm2 align REF.fasta FilteredBAM.bam FilteredMappingBAM.bam --preset CCS --sort
```

## Evaluation signals

- Every retained read carries the kinetic tags fi, ri, fp, rp; tag-absent reads are fully removed
- Distribution of Median_PW / Median_IPD in the filtered BAM lies within [0.3, 99th percentile of the original distribution]
- All retained reads satisfy fn >= 3 and rn >= 3; the filtered dataset contains >= 60,000 reads for full kinetic analyses
- No retained read is shorter than 500 bp or contains fewer than 100 A+Z bases when used for Z/A classification
- Downstream 21-mer context counts reach >= 10 observations per condition per k-mer, indicating the filter did not over-prune coverage

## Limitations

- The SVM read classifier downstream is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures, so filtering alone cannot rescue diluted samples
- Z-induced PW shifts (0.5–1.5-fold) overlap substantially with canonical Adenine variance in noisy kinetic measurements; filtering removes gross outliers but not this intrinsic overlap
- Reads shorter than 500 bp or with fewer than 100 A+Z bases are excluded due to limited A/Z content, which biases against short amplicons

## Evidence

- [readme] Required Tags: Kinetic signals (fi, ri, fp, rp) must be present.: "**Required Tags:** Kinetic signals (fi, ri, fp, rp) must be present."
- [methods] Signal quality cutoffs: "Reads with Median_PW / Median_IPD < 0.3 or > 99th percentile are discarded."
- [methods] Pass-count requirement: "Minimum 3 passes required for both forward (fn >= 3) and reverse (rn >= 3) strands."
- [discussion] Length and A+Z content gate: "Reads shorter than 500 bp or containing fewer than 100 A + Z bases were also excluded due to the limited number of A/Z bases"
- [discussion] Dataset-level quality threshold: "We maintained a data quality threshold of at least 60,000 reads per dataset with forward and reverse pass numbers no less than 3"
