---
name: rca-copy-number-stratified-analysis
description: Use when isoCirc raw nanopore reads and TRF-called consensus sequences
  are available and you need to verify consensus error-rate reduction stratified by
  RCA copy number, or to report average copy number and read-length statistics.
when_to_use_negative:
- You need to identify high-confidence BSJs or full-length circRNA isoforms from raw
  data — use the full isoCirc pipeline with its stringent false-positive filtering
  instead of this error-rate stratification
- Input consensus sequences were not called from RCA-derived tandem-repeat reads (e.g.
  single-pass long reads without rolling circle amplification), so copy number inferred
  as floor(raw length / consensus length) is meaningless
- You want splice-aware mapping of reads to the reference genome rather than read-to-consensus
  alignment for error-rate computation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: isoCirc
  role: Source pipeline that produced the raw nanopore reads and called consensus
    sequences with Tandem Repeats Finder using default parameters
  repo: https://github.com/Xinglab/isoCirc
- name: minimap2
  role: Aligns each raw read back to its consensus sequence to obtain per-base matches,
    mismatches, insertions, and deletions for error-rate computation
  repo: https://github.com/lh3/minimap2
- name: Python
  role: Used with pandas to bin reads by inferred RCA copy number, aggregate per-bin
    mean error rates, and write error_rates_by_copy_number.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:650dec9d6eff99b31e07c5202ea6c16bf75ad54e2094ea2e34dc948afe734a83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rca-copy-number-stratified-analysis

## Summary

Quantifies how consensus sequences called from isoCirc rolling-circle-amplified nanopore reads reduce error rate as a function of RCA copy number, by aligning each HEK293 raw read to its TRF-called consensus with minimap2 and binning per-base error rates (1 - identity) by inferred copy number (floor(raw length / consensus length)).

## When to use

Use when isoCirc raw nanopore reads and TRF-called consensus sequences are available and you need to verify consensus error-rate reduction stratified by RCA copy number, or to report average copy number and read-length statistics.

## When NOT to use

- You need to identify high-confidence BSJs or full-length circRNA isoforms from raw data — use the full isoCirc pipeline with its stringent false-positive filtering instead of this error-rate stratification
- Input consensus sequences were not called from RCA-derived tandem-repeat reads (e.g. single-pass long reads without rolling circle amplification), so copy number inferred as floor(raw length / consensus length) is meaningless
- You want splice-aware mapping of reads to the reference genome rather than read-to-consensus alignment for error-rate computation

## Inputs

- HEK293 isoCirc raw nanopore reads (FASTA/FASTQ)
- TRF-called consensus sequences generated with isoCirc default TRF parameters (match 2, mismatch 7, indel 7, match probability 80, indel probability 10, minimum alignment score 100, maximum period size 2000)

## Outputs

- Stratified error-rate table (error_rates_by_copy_number.csv) with per-copy-number-bin mean error rates
- Summary statistics: average RCA copy number (~14.5), average raw-read length (~5,679 nt), average consensus length (~460 nt)

## How to apply

Load the HEK293 isoCirc raw nanopore reads (FASTA/FASTQ) together with the consensus sequences called by Tandem Repeats Finder under the isoCirc defaults (match 2, mismatch 7, indel 7, match probability 80, indel probability 10, minimum alignment score 100, maximum period size 2000). Align each raw read back to its consensus using minimap2 (>= 2.11) to extract per-base matches, mismatches, insertions, and deletions, and compute per-read error rate as 1 - identity. Infer each read's RCA copy number as floor(raw read length / consensus length), then bin reads by copy number and aggregate per-bin mean error rates with Python/pandas, alongside summary values for average copy number, average raw-read length, and average consensus length. Save the stratified table to error_rates_by_copy_number.csv and check that error rate decreases with increasing copy number, since consensus from reads with more than 10 copies carries the lowest error rate (Supplementary Fig. 1).

## Related tools

- **isoCirc** (Source pipeline that produced the raw nanopore reads and called consensus sequences with Tandem Repeats Finder using default parameters) — https://github.com/Xinglab/isoCirc
- **minimap2** (Aligns each raw read back to its consensus sequence to obtain per-base matches, mismatches, insertions, and deletions for error-rate computation) — https://github.com/lh3/minimap2
- **Python** (Used with pandas to bin reads by inferred RCA copy number, aggregate per-bin mean error rates, and write error_rates_by_copy_number.csv)

## Examples

```
minimap2 -a consensus.fa raw_reads.fq > raw_to_consensus.sam
```

## Evaluation signals

- Error rate decreases monotonically with increasing RCA copy number, with consensus from reads having more than 10 copies showing the lowest error rate (Supplementary Fig. 1)
- Summary values approximate the reported averages: copy number ~14.5, raw reads ~5,679 nt, consensus ~460 nt
- Per-read copy number equals floor(raw read length / consensus length) and per-read error rate equals 1 - identity from the minimap2 alignment
- Output file error_rates_by_copy_number.csv exists with per-bin mean error rates and summary statistics

## Limitations

- Copy number is inferred by length ratio rather than measured directly from the tandem-repeat structure, so partial/uneven spans or alignment indels can bias per-read copy-number estimates
- The validation target (monotonic error-rate decrease, Supplementary Fig. 1) was established on HEK293 isoCirc data; other cell types or datasets may not reproduce the same summary averages
- Consensus calling itself depends on TRF parameters (e.g. maximum period size 2000); deviating from isoCirc defaults changes the consensus sequences fed into this analysis

## Evidence

- [methods] Consensus calling with Tandem Repeats Finder (TRF)): "Consensus calling with Tandem Repeats Finder (TRF))"
- [discussion] circular consensus improves accuracy: "circular consensus sequencing is a well-established approach to improve the accuracy of PacBio long reads"
- [discussion] After calling consensus sequences from isoCirc raw reads, we use stringent criteria: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [methods] using rolling circle amplification (RCA) followed by long-read sequencing: "using rolling circle amplification (RCA) followed by long-read sequencing"
- [readme] minimap2 requirement in README: "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
