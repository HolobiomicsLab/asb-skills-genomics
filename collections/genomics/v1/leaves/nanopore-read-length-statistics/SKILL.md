---
name: nanopore-read-length-statistics
description: Use when characterizing a nanopore WGS dataset by read-length statistics
  and genome-wide coverage at MAPQ ≥ 20 across alternative aligners (e.g., rebuilding
  the NA19240 control metrics from Table 2).
when_to_use_negative:
- Input is not whole-genome nanopore data (e.g., targeted panels or short reads) —
  genome-wide coverage fractions and long-read length statistics will not be comparable
  to Table 2.
- The goal is SV calling or coverage-based polishing/filtering of calls (disCoverage)
  rather than dataset-level QC metrics.
- Comparing aligner call sets rather than characterizing the underlying read set and
  alignment coverage — this skill produces only QC/coverage summaries, not variant
  calls.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: NanoStat
  role: Computes read-length statistics (mean read length, targeting 14,565 bp) from
    the NA19240 FASTQ reads.
- name: mosdepth
  role: Computes per-BAM genome-wide coverage distributions and summary depth per
    chromosome with MAPQ ≥ 20 filtering, yielding the genome-covered fractions and
    median coverage.
  repo: https://github.com/brentp/mosdepth
- name: minimap2
  role: One of the three aligners whose BAM is used to derive a per-aligner MAPQ ≥
    20 coverage fraction.
- name: NGMLR
  role: One of the three aligners whose BAM is used to derive a per-aligner MAPQ ≥
    20 coverage fraction.
- name: lra
  role: One of the three aligners whose BAM is used to derive a per-aligner MAPQ ≥
    20 coverage fraction.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:d9451e89d7cc373dc189aca60ec3a04fd67d77f3f9ddf1dc17ab2b72e52d54d6
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nanopore-read-length-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces baseline nanopore WGS dataset metrics — mean read length via NanoStat on the FASTQ and MAPQ ≥ 20 genome-wide coverage fractions via mosdepth on each aligner BAM — for the NA19240 control dataset, mirroring the Table 2 row (14,565 bp; 93/92/91% for lra/minimap2/NGMLR; median 16/17/15×).

## When to use

Use when characterizing a nanopore WGS dataset by read-length statistics and genome-wide coverage at MAPQ ≥ 20 across alternative aligners (e.g., rebuilding the NA19240 control metrics from Table 2).

## When NOT to use

- Input is not whole-genome nanopore data (e.g., targeted panels or short reads) — genome-wide coverage fractions and long-read length statistics will not be comparable to Table 2.
- The goal is SV calling or coverage-based polishing/filtering of calls (disCoverage) rather than dataset-level QC metrics.
- Comparing aligner call sets rather than characterizing the underlying read set and alignment coverage — this skill produces only QC/coverage summaries, not variant calls.

## Inputs

- NA19240 nanopore FASTQ reads (PromethION WGS)
- BAM alignments of the NA19240 reads produced by minimap2, NGMLR, and lra
- Reference genome used for alignment (hg38, per the article's reference-genome comparison)

## Outputs

- Per-aligner fraction of the genome covered at MAPQ ≥ 20 (93/92/91% for lra/minimap2/NGMLR)
- Median genome-wide coverage per aligner (16/17/15×)
- Mean read length statistic from NanoStat (target 14,565 bp)
- Summary table na19240_read_stats_coverage.tsv reproducing the NA19240 row of Table 2

## How to apply

Obtain the publicly available NA19240 PromethION nanopore FASTQ reads together with the three BAM alignments generated with minimap2, NGMLR, and lra. Run NanoStat on the FASTQ to derive read-length statistics, targeting the reported mean read length of 14,565 bp. Then run mosdepth separately on each of the three BAMs with the mapping-quality flag set so only reads with MAPQ ≥ 20 contribute depth, extracting the fraction of the genome covered and the median coverage per aligner. Compile the mean read length and the three per-aligner coverage percentages (and median coverage 16/17/15× for lra/minimap2/NGMLR) into a single summary table, na19240_read_stats_coverage.tsv. Validation: the tabulated values must reproduce the NA19240 row of Table 2 within rounding tolerance, since these metrics characterize the control sample used to assess aligner/variant-caller agreement.

## Related tools

- **NanoStat** (Computes read-length statistics (mean read length, targeting 14,565 bp) from the NA19240 FASTQ reads.)
- **mosdepth** (Computes per-BAM genome-wide coverage distributions and summary depth per chromosome with MAPQ ≥ 20 filtering, yielding the genome-covered fractions and median coverage.) — https://github.com/brentp/mosdepth
- **minimap2** (One of the three aligners whose BAM is used to derive a per-aligner MAPQ ≥ 20 coverage fraction.)
- **NGMLR** (One of the three aligners whose BAM is used to derive a per-aligner MAPQ ≥ 20 coverage fraction.)
- **lra** (One of the three aligners whose BAM is used to derive a per-aligner MAPQ ≥ 20 coverage fraction.)

## Examples

```
mosdepth -Q 20 -n --fast-mode na19240_lra na19240.lra.bam
```

## Evaluation signals

- NanoStat mean read length on the NA19240 FASTQ equals 14,565 bp within rounding tolerance.
- mosdepth with MAPQ ≥ 20 gives genome-covered fractions of 93/92/91% for lra/minimap2/NGMLR BAMs (rounding tolerance).
- Median genome-wide coverage is 16/17/15× for lra/minimap2/NGMLR, matching Table 2.
- The compiled na19240_read_stats_coverage.tsv reproduces the NA19240 row of Table 2 (mean read length plus three coverage percentages).
- Each mosdepth run used the mapping-quality threshold option so reads below MAPQ 20 were excluded from depth calculations.

## Limitations

- No software versions were reported for the aligners (lra, minimap2, NGMLR), so exact reproduction may vary with tool version.
- NGMLR is slower and had known bugs at the time of writing, which can affect its alignment-based coverage metrics.
- Genome coverage is not uniform — local peaks and valleys occur without necessarily indicating an SV, and unresolved genome regions limit coverage-based interpretation.
- Coverage analysis applies only to deletions and duplications, not all SV types.

## Evidence

- [full_text] The NA19240 control dataset had a mean read length of 14,565 bp, with 93/92/91% of the genome covered (MAPQ ≥ 20) using lra/minimap2/NGMLR: [evidence span withheld — anchor status: not_found]
- [readme] mapping quality threshold. reads with a quality less than this value are ignored: [evidence span withheld — anchor status: external_doc]
- [readme] a distribution of proportion of bases covered at or above a given threshold for each chromosome and genome-wide.: [evidence span withheld — anchor status: external_doc]
