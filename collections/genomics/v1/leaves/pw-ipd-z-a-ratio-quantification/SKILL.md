---
name: pw-ipd-z-a-ratio-quantification
description: Use when comparing matched dZ-DNA and native PacBio CCS datasets (with
  --keep-kinetics) to test whether A-to-Z substitution alters pulse width (PW) or
  inter-pulse duration (IPD) in a specific sequence context or motif family.
when_to_use_negative:
- The reads lack PacBio kinetic tags (no fi/ri/fp/rp), e.g., CCS BAMs generated without
  --keep-kinetics — kinetic fold-change quantification is impossible.
- The goal is simply to detect or call Z bases (dZ-DNA classification, A/Z calling)
  rather than quantify kinetic signal changes — use z-calling-base/z-calling-read
  instead.
- Contexts contain multiple adjacent A/Z bases within the 21-mer window, where adjacent-A/Z
  interference invalidates per-base ratio attribution.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: Provides filter_bam.py (kinetic/quality read filtering) and z-bam2txt (BAM-to-per-base
    table extraction of PW/IPD signals)
  repo: https://github.com/xiaochuanle/Z-Calling
- name: pbmm2
  role: Aligns filtered CCS reads to the reference genome (--preset CCS --sort) before
    kinetic signal extraction
- name: Python (pandas/numpy)
  role: Reconstructs 21-mer contexts, aggregates forward-strand PW/IPD values, and
    computes mean ± SD Z/A fold-change ratios
- name: samtools
  role: BAM manipulation used in the Z-Calling pipeline (e.g., samtools fasta conversion
    in downstream analysis)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:88e984e2c0b7bfec8b5a433d1509930731be1ecf6a66e71432747526198901e6
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# pw-ipd-z-a-ratio-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies sequence-context-dependent effects of dA/dZ (2,6-diaminopurine) substitution on PacBio polymerase kinetics by computing per-motif PW and IPD Z/A fold-change ratios from HiFi reads processed with Z-Calling. It reproduces findings such as the 1.46-fold PW elongation in TTZTT vs TTATT and 0.84-fold shortening in TTZGG vs TTAGG.

## When to use

Use when comparing matched dZ-DNA and native PacBio CCS datasets (with --keep-kinetics) to test whether A-to-Z substitution alters pulse width (PW) or inter-pulse duration (IPD) in a specific sequence context or motif family.

## When NOT to use

- The reads lack PacBio kinetic tags (no fi/ri/fp/rp), e.g., CCS BAMs generated without --keep-kinetics — kinetic fold-change quantification is impossible.
- The goal is simply to detect or call Z bases (dZ-DNA classification, A/Z calling) rather than quantify kinetic signal changes — use z-calling-base/z-calling-read instead.
- Contexts contain multiple adjacent A/Z bases within the 21-mer window, where adjacent-A/Z interference invalidates per-base ratio attribution.

## Inputs

- PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp) from dZ-DNA and matched native DNA amplicons (generated with cc --keep-kinetics)
- Reference genome FASTA (e.g., E. coli) used for pbmm2 alignment
- Motif definitions / target positions (e.g., 21-mers centered on the A/Z base, positions TTZTT vs TTATT)

## Outputs

- Per-base probability/text table from z-bam2txt (QueryName, ReadID, Strand, QueryPos, Base, RefName, RefPos, Probability for mapped reads)
- motif_kinetic_ratios.tsv: mean ± SD PW and IPD Z/A fold-change ratios per motif family
- Fold-change values interpreted against the reported 0.5–1.5-fold Z-induced kinetic shift range (e.g., 1.46 ± 0.12 for TTZTT; 0.84 ± 0.08 for TTZGG at base position 12)

## How to apply

Filter raw dZ-DNA and native BAMs with Z-Calling's filter_bam.py, requiring kinetic tags (fi, ri, fp, rp), a median PW/median IPD ratio >=0.3 and below the 99th percentile, and >=3 passes on both strands (fn>=3, rn>=3). Align filtered reads with pbmm2 (pbmm2 align REF.fasta FilteredBAM --preset CCS --sort), then convert the mapped BAM to a per-base probability table with 'z-bam2txt -zp -t 16', supplying the exact reference FASTA used for alignment so coordinates match. Reconstruct 21-mer contexts centered on each A/Z base, retain only contexts containing a single A/Z target (to avoid interference among adjacent A/Zs) with at least 10 observations in each condition, and aggregate forward-strand PW and IPD values for motif families of interest (e.g., TTZTT/TTATT, TTZGG/TTAGG). Compute mean ± SD Z/A fold-change ratios per motif with pandas/numpy and write a TSV (e.g., motif_kinetic_ratios.tsv).

## Related tools

- **Z-Calling** (Provides filter_bam.py (kinetic/quality read filtering) and z-bam2txt (BAM-to-per-base table extraction of PW/IPD signals)) — https://github.com/xiaochuanle/Z-Calling
- **pbmm2** (Aligns filtered CCS reads to the reference genome (--preset CCS --sort) before kinetic signal extraction)
- **Python (pandas/numpy)** (Reconstructs 21-mer contexts, aggregates forward-strand PW/IPD values, and computes mean ± SD Z/A fold-change ratios)
- **samtools** (BAM manipulation used in the Z-Calling pipeline (e.g., samtools fasta conversion in downstream analysis))

## Examples

```
python py filter_bam.py -b raw.bam -o filtered.bam && pbmm2 align REF.fasta filtered.bam filtered.mapped.bam --preset CCS --sort && build/z-bam2txt -zp -t 16 filtered.mapped.bam REF.fasta > table.tsv
```

## Evaluation signals

- PW Z/A fold-change matches reported values: 1.46 ± 0.12 (p = 2.75e−306) for TTZTT vs TTATT and 0.84 ± 0.08 (p = 3.95e−48) for TTZGG vs TTAGG at base position 12
- All computed Z/A ratios fall within the reported 0.5–1.5-fold Z-induced kinetic shift range
- Every retained context 21-mer contains exactly one A/Z target base and has ≥10 observations in both dZ and native conditions
- Retained reads satisfy the filter criteria: kinetic tags present, median PW/median IPD ratio ≥0.3 and below the 99th percentile, fn ≥3 and rn ≥3 passes
- z-bam2txt output was produced with the exact reference FASTA used for pbmm2 alignment so RefPos coordinates are consistent

## Limitations

- The subtle Z-induced kinetic shift (0.5–1.5-fold) overlaps substantially with canonical adenine variance due to inherent PacBio kinetic measurement noise, limiting quantification in hybrid samples.
- Only context k-mers containing a single A/Z target can be analyzed; adjacent A/Zs interfere with per-base interpretation.
- Z incorporation affects PW primarily in a sequence-dependent manner — a ratio observed in one motif (e.g., TTZTT) may not transfer to another context (e.g., TTZGG, where PW is shortened instead).

## Evidence

- [results] context-dependent PW finding: [evidence span withheld — anchor status: not_found]
