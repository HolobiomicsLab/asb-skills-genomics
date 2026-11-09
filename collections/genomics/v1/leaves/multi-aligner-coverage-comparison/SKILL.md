---
name: multi-aligner-coverage-comparison
description: Use when a nanopore FASTQ dataset has been aligned with multiple aligners
  and you need per-aligner genome coverage fractions (MAPQ ≥ 20) and read-length statistics
  to compare alignment pipelines.
when_to_use_negative:
- You need per-sample SV call polishing rather than aggregate sequencing/coverage
  metrics — disCoverage-based coverage filtering is the appropriate skill there
- Only a single alignment exists and no cross-aligner comparison is intended; a single
  mosdepth run suffices without the multi-aligner compilation step
- The reads are short-read Illumina data; the reported read-length statistics (mean
  14,565 bp) and pipeline context are specific to long nanopore reads
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mosdepth
  role: Computes per-base/summary depth from each aligner's BAM with MAPQ filtering
    (-Q 20) to obtain genome coverage fractions and median coverage
  repo: https://github.com/brentp/mosdepth
- name: NanoStat
  role: Derives read-length statistics (mean read length) from the NA19240 nanopore
    FASTQ reads
- name: minimap2
  role: One of the three aligners whose BAM output feeds the coverage comparison
- name: NGMLR
  role: One of the three aligners whose BAM output feeds the coverage comparison
- name: lra
  role: One of the three aligners whose BAM output feeds the coverage comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:0939ae468c224fd1a7531b7aa7c4689b022253c021b7b0d2803d3a0d7e808e9c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/brentp/mosdepth
  tools:
  - NGMLR
  - NanoStat
  - lra
  - minimap2
  - mosdepth
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-aligner-coverage-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Characterizes a nanopore WGS sample by running read-length statistics with NanoStat and computing MAPQ-filtered genome coverage with mosdepth across BAMs produced by three aligners (minimap2, NGMLR, lra), enabling a per-aligner coverage comparison such as the NA19240 control row of the paper's Table 2.

## When to use

Use when a nanopore FASTQ dataset has been aligned with multiple aligners and you need per-aligner genome coverage fractions (MAPQ ≥ 20) and read-length statistics to compare alignment pipelines.

## When NOT to use

- You need per-sample SV call polishing rather than aggregate sequencing/coverage metrics — disCoverage-based coverage filtering is the appropriate skill there
- Only a single alignment exists and no cross-aligner comparison is intended; a single mosdepth run suffices without the multi-aligner compilation step
- The reads are short-read Illumina data; the reported read-length statistics (mean 14,565 bp) and pipeline context are specific to long nanopore reads

## Inputs

- NA19240 nanopore FASTQ reads (publicly available control dataset)
- BAM alignments of the same reads produced by minimap2, NGMLR, and lra against the chosen reference (hg38 in the paper's preferred setup)

## Outputs

- NanoStat read-length statistics with mean read length (target: 14,565 bp)
- Per-aligner genome coverage summary: fraction of genome covered at MAPQ ≥ 20 (target: 93/92/91%) and median coverage (16/17/15×)
- Combined summary table na19240_read_stats_coverage.tsv

## How to apply

Obtain the publicly available NA19240 nanopore reads and the three BAM alignments generated with minimap2, NGMLR, and lra. Run NanoStat on the NA19240 FASTQ reads to derive read-length statistics, targeting the reported mean read length of 14,565 bp. Run mosdepth separately on each of the three BAMs with a mapping-quality threshold of 20 (-Q/--mapq 20) so only quality-filtered alignments contribute to depth; use mosdepth's distribution/threshold outputs to compute the fraction of the genome covered per aligner. Extract the per-aligner fraction of genome covered, targeting the reported 93/92/91% values, and the median coverage of 16/17/15×. Compile the mean read length with the three per-aligner coverage percentages into a single summary table (na19240_read_stats_coverage.tsv), validating that values reproduce the NA19240 row of Table 2 within rounding tolerance.

## Related tools

- **mosdepth** (Computes per-base/summary depth from each aligner's BAM with MAPQ filtering (-Q 20) to obtain genome coverage fractions and median coverage) — https://github.com/brentp/mosdepth
- **NanoStat** (Derives read-length statistics (mean read length) from the NA19240 nanopore FASTQ reads)
- **minimap2** (One of the three aligners whose BAM output feeds the coverage comparison)
- **NGMLR** (One of the three aligners whose BAM output feeds the coverage comparison)
- **lra** (One of the three aligners whose BAM output feeds the coverage comparison)

## Examples

```
mosdepth --mapq 20 --by 500 na19240_minimap2 na19240.minimap2.sorted.bam
```

## Evaluation signals

- NanoStat output yields a mean read length of 14,565 bp (within rounding tolerance) for NA19240
- Genome coverage fractions at MAPQ ≥ 20 match Table 2: 93/92/91% for lra/minimap2/NGMLR
- Median coverage per aligner matches the reported 16/17/15×
- The compiled na19240_read_stats_coverage.tsv reproduces the NA19240 row of Table 2 as a whole
- Each mosdepth run used -Q 20 so coverage derives only from quality-filtered alignments, verifiable from the command logs/flags

## Limitations

- Coverage is not uniform across a sample; local peaks and valleys occur without necessarily indicating an SV, so coverage metrics describe the genome aggregate only
- No software versions were reported for the aligners or callers, so exact version-dependent behavior cannot be pinned down
- Values are validated only to rounding tolerance against the published Table 2 row

## Evidence

- [readme] mosdepth MAPQ threshold flag used for the ≥ 20 coverage filtering: [evidence span withheld — anchor status: external_doc]
