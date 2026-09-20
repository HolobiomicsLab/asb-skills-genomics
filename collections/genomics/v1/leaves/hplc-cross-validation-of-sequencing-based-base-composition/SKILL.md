---
name: hplc-cross-validation-of-sequencing-based-base-composition
description: Use when a tool-based base-composition estimate (e.g., Z-Calling's aggregate
  Z/(A+Z) ratio over chromosomes/plasmids) must be cross-checked against an orthogonal
  nucleotide-composition measurement such as HPLC-UV on enzymatically digested genomic
  DNA.
when_to_use_negative:
- The sample contains no modified-base fraction to quantify, or HPLC-UV nucleotide-composition
  data cannot be generated for the same DNA extract
- The goal is locus-level rather than composition-level validation — HPLC measures
  bulk nucleotide composition and cannot pinpoint Z locations in sequences
- Input is already an HPLC chromatogram or a feature table of nucleoside abundances;
  no sequencing-based calling step is involved
tools:
- name: Z-Calling (z-calling-base, z-bam2txt, zfreq)
  role: Produces the sequencing-derived per-coordinate Z/(A+Z) ratios that are aggregated
    and compared to the HPLC-UV value
  repo: https://github.com/xiaochuanle/Z-Calling
- name: HPLC-UV
  role: Provides the independent nucleotide-composition reference measurement (Z/(A+Z))
    of enzymatically digested genomic DNA
- name: pbmm2
  role: Aligns filtered CCS reads (--preset CCS --sort) to the combined yeast+plasmid
    reference so per-coordinate Z ratios map to the correct contigs
- name: samtools
  role: BAM manipulation in the base-calling pipeline preceding the composition comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:1814dcaf913c6fbc35f1c8cf91ce1cef89f964a0cbca91527b0fd2763b41e4fb
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
  - HPLC-UV
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling (z-calling-base, z-bam2txt, zfreq)
  - pbmm2
  - samtools
  - z-bam2txt
  - z-calling-base
  - zfreq
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hplc-cross-validation-of-sequencing-based-base-composition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a sequencing-derived base-composition estimate (e.g., the genome-wide Z/(A+Z) fraction called by Z-Calling on PacBio HiFi reads) against an independent HPLC-UV nucleotide-composition measurement, as done for transformed yeast hybrid DNA where 26.4% called Z matched the 24.7% HPLC-UV value.

## When to use

Use when a tool-based base-composition estimate (e.g., Z-Calling's aggregate Z/(A+Z) ratio over chromosomes/plasmids) must be cross-checked against an orthogonal nucleotide-composition measurement such as HPLC-UV on enzymatically digested genomic DNA.

## When NOT to use

- The sample contains no modified-base fraction to quantify, or HPLC-UV nucleotide-composition data cannot be generated for the same DNA extract
- The goal is locus-level rather than composition-level validation — HPLC measures bulk nucleotide composition and cannot pinpoint Z locations in sequences
- Input is already an HPLC chromatogram or a feature table of nucleoside abundances; no sequencing-based calling step is involved

## Inputs

- Aggregated per-coordinate Z-ratio table from zfreq (chr, coor, num_records_above_threshold, num_records_below_threshold, ratio_above_threshold)
- ZP-tagged BAM from z-calling-base (k11-mixed-AZ model) for the same sample
- Enzymatically digested genomic DNA from the same transformed yeast sample (genome + 2-micron and engineered plasmids) for HPLC-UV analysis
- Reference HPLC-UV nucleotide-composition value (e.g., 24.7% Z/(A+Z))

## Outputs

- Aggregate genomic Z/(A+Z) fraction estimated by Z-Calling (e.g., 26.4%)
- Cross-validation comparison of sequencing-derived vs. HPLC-UV Z/(A+Z) fractions with an accept/reject judgment

## How to apply

Generate the sequencing-side composition by running Z-Calling's A/Z base-calling module (build/z-calling-base -k 11 with model/k11-mixed-AZ/scripted_m11.pth) on filtered PacBio CCS reads, converting the ZP-tagged BAM with z-bam2txt, and aggregating per-coordinate Z ratios with build/zfreq, which computes ratio_above_threshold as Z count / (A + Z count) per chromosome/contig coordinate. Independently, digest the genomic DNA from the same sample (here, transformed S. cerevisiae S288C carrying pRS426-ApPurZ-ApdATPase and pRS425-ApDUF550 for dZTP biosynthesis) enzymatically and measure the Z/(A+Z) nucleotide fraction by HPLC-UV, since HPLC 'measures nucleotide composition'. Compare the aggregate sequencing-derived fraction (e.g., ~26.4%) to the HPLC-UV value (e.g., 24.7%); accept the sequencing result if the two estimates fall close to each other. The rationale is that HPLC quantifies total modified-nucleotide abundance without sequence resolution, while Z-Calling provides the single-nucleotide-resolved estimate that HPLC cannot.

## Related tools

- **Z-Calling (z-calling-base, z-bam2txt, zfreq)** (Produces the sequencing-derived per-coordinate Z/(A+Z) ratios that are aggregated and compared to the HPLC-UV value) — https://github.com/xiaochuanle/Z-Calling
- **HPLC-UV** (Provides the independent nucleotide-composition reference measurement (Z/(A+Z)) of enzymatically digested genomic DNA)
- **pbmm2** (Aligns filtered CCS reads (--preset CCS --sort) to the combined yeast+plasmid reference so per-coordinate Z ratios map to the correct contigs)
- **samtools** (BAM manipulation in the base-calling pipeline preceding the composition comparison)

## Examples

```
build/zfreq -i Result.tsv -o Output_Freq.tsv
```

## Evaluation signals

- The aggregate Z-Calling Z/(A+Z) fraction (e.g., ~26.4%) falls close to the independent HPLC-UV value (e.g., 24.7%) for the same sample
- The zfreq output table contains exactly the columns chr, coor, num_records_above_threshold, num_records_below_threshold, and ratio_above_threshold
- ratio_above_threshold values lie in [0, 1] for every coordinate and equal num_records_above_threshold / (num_records_above_threshold + num_records_below_threshold)
- Ratios are reported across all expected contigs: yeast chromosomes, the 2-micron plasmid, and plasmids pRS426-ApPurZ-ApdATPase and pRS425-ApDUF550

## Limitations

- HPLC quantifies nucleotide composition but cannot pinpoint Z locations in sequences, so agreement is at the aggregate-fraction level only
- The subtle Z-induced PacBio kinetic shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance, limiting hybrid-sample quantification precision
- Estimates depend on upstream filtering (median PW/median IPD in [0.3, 99th percentile], fn >= 3 and rn >= 3 passes); overly aggressive filtering could bias the aggregate fraction

## Evidence

- [full_text] Z-Calling estimated that 26.4% of bases paired with thymine were Z bases in the transformed yeast hybrid DNA dataset, closely matching the 24.7% Z/(A+Z) value independently measured by HPLC-UV: [evidence span withheld — anchor status: not_found]
