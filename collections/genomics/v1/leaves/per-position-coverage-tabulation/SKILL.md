---
name: per-position-coverage-tabulation
description: Use when you have MAPQ/flag-filtered read alignments (BAM) and need per-base
  counts of soft/hard-clipped reads alongside total coverage to locate candidate assembly-error
  breakpoints or heterozygous positions.
when_to_use_negative:
- Alignments are unfiltered raw mapping output — clipping tallies on low-MAPQ, secondary,
  or duplicate records inflate false clip positions; apply the '-F 1796 -q 20' filter
  first
- You need final CRE/CSE error calls or AQI scores rather than the intermediate per-position
  clipping/coverage tables — this skill produces only the position-level clip counts
  and coverage
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: CRAQ
  role: Method whose first step extracts clipped reads (CIGAR S/H) from filtered BAMs
    and generates per-position clip-count/coverage tables (SR_clip.coverRate, LR_clip.coverRate)
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: SAMtools
  role: Filters alignments with 'samtools view -F 1796 -q 20' and provides BAM access
    for scanning CIGAR strings of mapped reads
  repo: https://github.com/lh3/minimap2
- name: Perl
  role: Implementation language (version >= 5) for CRAQ's clipping-extraction and
    tabulation scripts
  repo: https://github.com/JiaoLaboratory/CRAQ
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:83afbb2d37529c5148997d816efeb3cc3198548fc1f9d8313dae4963e71e7723
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Perl
  - SAMtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-position-coverage-tabulation

## Summary

Tabulates, for every genomic position in filtered NGS and SMS BAM alignments, the number of clipped reads (CIGAR S/H operations) and the total mapped-read coverage, producing CRAQ's SR_clip.coverRate- and LR_clip.coverRate-style tables. This per-position clipping/coverage tally is the first CRAQ step toward separating assembly error breakpoints from heterozygous loci.

## When to use

Use when you have MAPQ/flag-filtered read alignments (BAM) and need per-base counts of soft/hard-clipped reads alongside total coverage to locate candidate assembly-error breakpoints or heterozygous positions.

## When NOT to use

- Alignments are unfiltered raw mapping output — clipping tallies on low-MAPQ, secondary, or duplicate records inflate false clip positions; apply the '-F 1796 -q 20' filter first
- You need final CRE/CSE error calls or AQI scores rather than the intermediate per-position clipping/coverage tables — this skill produces only the position-level clip counts and coverage

## Inputs

- Filtered NGS short-read alignment BAM (SR_sort.bam; 'samtools view -F 1796 -q 20')
- Filtered SMS long-read alignment BAM (LR_sort.bam; same filter parameters)
- CIGAR strings of alignments (S/H operations) and MAPQ/flag-filtered read records

## Outputs

- Tab-delimited NGS clipping table (SR_clip.coverRate equivalent): chr, position, strand, number of clipped-reads, total coverage at that position
- Tab-delimited SMS clipping table (LR_clip.coverRate equivalent): chr, position, strand, number of clipped-reads, total coverage at that position

## How to apply

Start from CRAQ's pre-filtered alignments (SR_sort.bam for NGS short reads, LR_sort.bam for SMS long reads), produced with 'samtools view -F 1796 -q 20' to drop MAPQ<20, unmapped, secondary, QC-failed, and PCR-duplicate records. Iterate through each BAM with SAMtools/pysam and select every read whose CIGAR string contains an S (soft-clip) or H (hard-clip) operation, as CRAQ does for both libraries. For each clipped read, record the mapped reference base coordinate of the clip and encode clipping orientation as strand '+' for left-clipped and '-' for right-clipped. At each genomic position, tally the number of clipped reads and the total mapped-read coverage, then write two tab-delimited tables with columns chr, position, strand, number of clipped-reads, and total coverage at that position — one NGS table (SR_clip.coverRate equivalent) and one SMS table (LR_clip.coverRate equivalent). These clipping ratios and coverage values are subsequently compared against CRAQ cutoffs (heterozygous if within h = 0.4–0.6, breakpoint if above f = 0.75), so their accuracy directly determines downstream CRE/CSE and CRH/CSH calls.

## Related tools

- **CRAQ** (Method whose first step extracts clipped reads (CIGAR S/H) from filtered BAMs and generates per-position clip-count/coverage tables (SR_clip.coverRate, LR_clip.coverRate)) — https://github.com/JiaoLaboratory/CRAQ
- **SAMtools** (Filters alignments with 'samtools view -F 1796 -q 20' and provides BAM access for scanning CIGAR strings of mapped reads) — https://github.com/lh3/minimap2
- **Perl** (Implementation language (version >= 5) for CRAQ's clipping-extraction and tabulation scripts) — https://github.com/JiaoLaboratory/CRAQ

## Examples

```
samtools view -F 1796 -q 20 SR_sort.bam | python parse_clips.py
```

## Evaluation signals

- Both output tables (NGS and SMS) exist and contain exactly the five required columns: chr, position, strand, number of clipped-reads, total coverage
- At every position, reported clipped-read count never exceeds total coverage at that position
- Every tallied position derives from a read whose CIGAR string contains an S or H operation
- No tabulated record originates from an alignment failing the '-F 1796 -q 20' filter (MAPQ <20, unmapped, secondary, QC-failed, or PCR-duplicated)
- Clipping orientation encoding is consistent: '+' for left-clipped reads and '-' for right-clipped reads

## Limitations

- Per-position clip ratios can be confounded by sequencing errors or inaccurate read mapping — CRAQ notes 'The ratio for assembly errors can be lower than 100% in practice due to sequencing errors or inaccurate read mapping'
- Positions within the intermediate clipping-ratio range (above h, below f) are ambiguous heterozygous-or-error regions and are not definitively classified at this stage
- Read mapping (and hence the underlying BAM inputs) is CRAQ's most resource-intensive step; users may need to split query sequences into multiple fragments to parallelize

## Evidence

- [discussion] CIGAR S/H extraction from filtered BAMs: "CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string"
- [discussion] Alignment filtering parameters: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [discussion] Low-quality alignment removal criteria: [evidence span withheld — anchor status: not_found]
- [readme] Clipping-ratio-based variant coordinates (README): "It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped"
- [discussion] Downstream classification cutoffs on clip ratios: "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
