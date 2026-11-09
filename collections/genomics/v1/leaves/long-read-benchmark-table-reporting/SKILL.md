---
name: long-read-benchmark-table-reporting
description: Use when benchmarking long-read (nanopore PromethION) whole-genome data
  aligned with multiple aligners and you need comparable read-length statistics and
  MAPQ-filtered genome coverage fractions per aligner.
when_to_use_negative:
- When your goal is SV calling or coverage-based filtering of variant calls (use disCoverage
  on putative SV positions instead — it is not a benchmarking tool)
- When reads are short reads or targeted/exome data where genome-wide coverage fractions
  are not the comparable metric of interest
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: NanoStat
  role: Computes read-length statistics (mean 14,565 bp) from the NA19240 FASTQ reads
- name: mosdepth
  role: Calculates genome-wide coverage depth and fraction of genome covered at MAPQ
    ≥ 20 from each aligner's BAM
  repo: https://github.com/brentp/mosdepth
- name: minimap2
  role: One of the three aligners whose BAMs are benchmarked for coverage
- name: NGMLR
  role: One of the three aligners whose BAMs are benchmarked for coverage
- name: lra
  role: One of the three aligners whose BAMs are benchmarked for coverage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:c0b893c63035ae732dfecdf6c9919579644fdcf58a113fe7d1a63352dd562a03
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# long-read-benchmark-table-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the sequencing and coverage benchmark statistics of a nanopore control dataset — mean read length via NanoStat and MAPQ-filtered genome coverage via mosdepth — and compile them into a per-aligner summary table (e.g. Table 2 of the SERPINC1 nanopore study).

## When to use

Use when benchmarking long-read (nanopore PromethION) whole-genome data aligned with multiple aligners and you need comparable read-length statistics and MAPQ-filtered genome coverage fractions per aligner.

## When NOT to use

- When your goal is SV calling or coverage-based filtering of variant calls (use disCoverage on putative SV positions instead — it is not a benchmarking tool)
- When reads are short reads or targeted/exome data where genome-wide coverage fractions are not the comparable metric of interest

## Inputs

- NA19240 nanopore FASTQ reads (public PromethION whole-genome dataset)
- Three BAM alignment files of NA19240 reads (minimap2, NGMLR, and lra)
- Reference genome (hg38)

## Outputs

- NanoStat read-length report with mean read length (target: 14,565 bp)
- mosdepth coverage outputs per aligner (e.g. {prefix}.mosdepth.summary.txt, .global.dist.txt) computed at MAPQ ≥ 20
- Summary table na19240_read_stats_coverage.tsv with mean read length and 93/92/91% genome coverage per lra/minimap2/NGMLR alignment

## How to apply

Obtain the publicly available NA19240 nanopore FASTQ reads and the three BAM alignments generated with minimap2, NGMLR, and lra against the chosen reference (hg38 in this study). Run NanoStat on the FASTQ to derive read-length statistics, targeting the reported mean read length of 14,565 bp. Then run mosdepth separately on each of the three BAMs with MAPQ ≥ 20 filtering (mosdepth's -Q 20 flag ignores reads with mapping quality below the threshold), extracting the fraction of the genome covered and median coverage per aligner. The reported control values are 93/92/91% genome coverage and 16/17/15× median coverage for lra/minimap2/NGMLR. Compile mean read length plus the three per-aligner coverage percentages into a single TSV (na19240_read_stats_coverage.tsv) that reproduces the NA19240 row of Table 2 within rounding tolerance.

## Related tools

- **NanoStat** (Computes read-length statistics (mean 14,565 bp) from the NA19240 FASTQ reads)
- **mosdepth** (Calculates genome-wide coverage depth and fraction of genome covered at MAPQ ≥ 20 from each aligner's BAM) — https://github.com/brentp/mosdepth
- **minimap2** (One of the three aligners whose BAMs are benchmarked for coverage)
- **NGMLR** (One of the three aligners whose BAMs are benchmarked for coverage)
- **lra** (One of the three aligners whose BAMs are benchmarked for coverage)

## Examples

```
mosdepth -Q 20 -n -F 1796 na19240_minimap2 na19240.minimap2.bam
```

## Evaluation signals

- NanoStat mean read length matches 14,565 bp within rounding tolerance
- Per-aligner genome coverage at MAPQ ≥ 20 reproduces 93/92/91% (lra/minimap2/NGMLR) within rounding tolerance
- Median coverage per aligner reproduces 16/17/15× (lra/minimap2/NGMLR)
- mosdepth was run with -Q 20 so reads with MAPQ below 20 are excluded from depth calculation
- The final TSV reproduces the NA19240 row of Table 2 exactly (within rounding)

## Limitations

- No software versions reported for any of the aligners (lra, minimap2, NGMLR), so exact reproduction of alignment stats may vary with tool version
- Coverage metrics are only meaningful as relative benchmarks between aligners on the same reference; coverage is not uniform across a sample and local peaks/valleys occur without pointing to an SV
- These statistics describe the NA19240 control sample only; patient data (P1–7) are not publicly accessible due to NIHR BioResource restrictions

## Evidence

- [results] The NA19240 control dataset had a mean read length of 14,565 bp, with 93/92/91% of the genome covered (MAPQ ≥ 20) using lra/minimap2/NGMLR, and median coverage of 16/17/15×, respectively: [evidence span withheld — anchor status: not_found]
- [readme] mosdepth mapq flag: [evidence span withheld — anchor status: external_doc]
