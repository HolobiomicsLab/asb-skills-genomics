---
name: genome-wide-coverage-quantification
description: Use when characterizing a nanopore whole-genome dataset (e.g., FASTQ
  read lengths plus per-aligner BAM coverage) before or alongside structural-variant
  calling, to benchmark sequencing depth and alignment quality.
when_to_use_negative:
- Coverage-based filtering of SV calls (e.g., with disCoverage) is the wrong tool
  here — this skill characterizes global depth, it does not validate deletion/duplication
  calls against local coverage
- Single-locus or region-restricted depth queries (exome/targeted regions) rather
  than genome-wide nanopore WGS characterization
- Reads lack MAPQ-bearing alignments (raw FASTQ only) — mosdepth requires an aligned
  BAM/CRAM; use NanoStat-style read statistics only in that case
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: mosdepth
  role: Computes fast BAM/CRAM depth for WGS, including the proportion of bases covered
    at or above a threshold and mean depth, run per aligner with MAPQ ≥ 20 filtering
  repo: https://github.com/brentp/mosdepth
- name: NanoStat
  role: Derives read-length statistics (e.g., mean read length 14,565 bp for NA19240)
    from the nanopore FASTQ reads
- name: minimap2
  role: One of the three aligners whose BAM output is quantified for genome coverage
    in the comparison
- name: NGMLR
  role: One of the three aligners whose BAM output is quantified for genome coverage
    in the comparison
- name: lra
  role: One of the three aligners whose BAM output is quantified for genome coverage
    in the comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:60fe6e0b2ed64cf3476f07e955838aba49f323dc794b444e6e09e2d32bc44061
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
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# genome-wide-coverage-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies genome-wide sequencing coverage and read-length statistics for nanopore WGS data using NanoStat on FASTQ reads and mosdepth on BAM alignments with MAPQ ≥ 20 filtering, as done to characterize the NA19240 control dataset in this pipeline.

## When to use

Use when characterizing a nanopore whole-genome dataset (e.g., FASTQ read lengths plus per-aligner BAM coverage) before or alongside structural-variant calling, to benchmark sequencing depth and alignment quality.

## When NOT to use

- Coverage-based filtering of SV calls (e.g., with disCoverage) is the wrong tool here — this skill characterizes global depth, it does not validate deletion/duplication calls against local coverage
- Single-locus or region-restricted depth queries (exome/targeted regions) rather than genome-wide nanopore WGS characterization
- Reads lack MAPQ-bearing alignments (raw FASTQ only) — mosdepth requires an aligned BAM/CRAM; use NanoStat-style read statistics only in that case

## Inputs

- Nanopore FASTQ reads (e.g., the publicly available NA19240 PromethION dataset)
- BAM alignments produced with minimap2, NGMLR, and lra against a human reference (hg38 preferred for SV calling per the article)
- Reference genome FASTA if CRAM inputs are used

## Outputs

- Read-length statistics table (e.g., mean read length in bp from NanoStat)
- Per-aligner genome-wide coverage metrics: fraction of genome covered at MAPQ ≥ 20 and median coverage (e.g., 93/92/91%, 16/17/15×)
- Combined summary table such as na19240_read_stats_coverage.tsv

## How to apply

First run NanoStat on the nanopore FASTQ reads to obtain read-length statistics (the NA19240 control yielded a mean read length of 14,565 bp). Then run mosdepth separately on each BAM alignment — the pipeline compared minimap2, NGMLR, and lra alignments — using the map-quality threshold flag (-Q 20) so only reads with MAPQ ≥ 20 contribute to depth. Use mosdepth's distribution output to compute the fraction of the genome covered at or above the threshold and the median coverage; for NA19240 this gave 93/92/91% genome coverage (MAPQ ≥ 20) for lra/minimap2/NGMLR and median coverage of 16/17/15×. Compile the mean read length and the per-aligner coverage percentages into a single summary table for comparison against the published values.

## Related tools

- **mosdepth** (Computes fast BAM/CRAM depth for WGS, including the proportion of bases covered at or above a threshold and mean depth, run per aligner with MAPQ ≥ 20 filtering) — https://github.com/brentp/mosdepth
- **NanoStat** (Derives read-length statistics (e.g., mean read length 14,565 bp for NA19240) from the nanopore FASTQ reads)
- **minimap2** (One of the three aligners whose BAM output is quantified for genome coverage in the comparison)
- **NGMLR** (One of the three aligners whose BAM output is quantified for genome coverage in the comparison)
- **lra** (One of the three aligners whose BAM output is quantified for genome coverage in the comparison)

## Examples

```
mosdepth -Q 20 -n --fast-mode na19240_lra na19240.lra.lra.bam
```

## Evaluation signals

- Mean read length from NanoStat on NA19240 must equal 14,565 bp within rounding tolerance
- Genome coverage at MAPQ ≥ 20 must reproduce the Table 2 row of 93/92/91% for the minimap2/NGMLR/lra alignments
- Median coverage must fall at 16/17/15× for the lra/minimap2/NGMLR alignments, respectively
- Per-aligner coverage percentages must differ by only ~1–2 points across aligners (large deviations suggest a filtering or threshold error)
- Summary table (e.g., na19240_read_stats_coverage.tsv) must contain one coverage value per aligner plus the read-length statistic, with column semantics matching Table 2

## Limitations

- No software versions were reported for the aligners, so exact reproducibility of coverage numbers may vary with tool versions
- Coverage is not uniform in a sample: local peaks and valleys frequently appear without indicating an SV, and unresolved genome regions and unclear shift boundaries limit coverage-based interpretation
- Coverage-based analysis applies only to deletions and duplications, not to all SV types
- Values must match within rounding tolerance; published percentages are rounded to whole numbers

## Evidence

- [results] The NA19240 control dataset had a mean read length of 14,565 bp, with 93/92/91% of the genome covered (MAPQ ≥ 20) using lra/minimap2/NGMLR, and median coverage of 16/17/15×: [evidence span withheld — anchor status: not_found]
- [readme] Q --mapq <mapq>                  mapping quality threshold. reads with a quality less than this value are ignored [default: 0]: [evidence span withheld — anchor status: external_doc]
- [readme] a distribution of proportion of bases covered at or above a given threshold for each chromosome and genome-wide.: [evidence span withheld — anchor status: external_doc]
