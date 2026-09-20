---
name: bam-quality-filtered-coverage-computation
description: Use when you have one or more aligned BAM files (e.g. nanopore reads
  aligned by minimap2, NGMLR, or lra) and need MAPQ-thresholded genome coverage fractions,
  median depth, or per-base depth to characterize a dataset or support coverage-based
  SV refinement.
when_to_use_negative:
- Input is raw FASTQ with no alignment — read QC/length statistics require NanoStat,
  not coverage computation; a BAM/CRAM is required.
- You need to call SVs or validate SV types other than deletions and duplications
  — coverage analysis in this paper only supports filtering/polishing of deletions
  and duplications, and disCoverage is not a variant caller.
- Coverage in the sample is highly non-uniform with local peaks and valleys — coverage
  alone does not prove an SV, so do not use it as sole evidence for variant detection.
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: mosdepth
  role: Computes fast BAM/CRAM depth, distribution of bases covered at or above thresholds,
    and per-chromosome/genome-wide coverage summaries under a MAPQ filter (-Q 20)
    for each aligner's BAM
  repo: https://github.com/brentp/mosdepth
- name: minimap2
  role: One of the three aligners whose BAM output is the coverage input (NGMLR and
    lra are the other two)
- name: NanoStat
  role: Derives read-length statistics (mean 14,565 bp) from the NA19240 FASTQ reads
    to accompany the coverage metrics in the summary table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:6b15cfadc90e320aba3fe52e0b830a6a1be649a2fa06aa91df55758457385699
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
  - NanoStat
  - minimap2
  - mosdepth
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bam-quality-filtered-coverage-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes quality-filtered genome-wide coverage statistics from nanopore BAM alignments using mosdepth, e.g. reproducing the NA19240 control dataset's 93/92/91% genome coverage (MAPQ ≥ 20) and 16/17/15× median coverage for the lra/minimap2/NGMLR alignments (Table 2).

## When to use

Use when you have one or more aligned BAM files (e.g. nanopore reads aligned by minimap2, NGMLR, or lra) and need MAPQ-thresholded genome coverage fractions, median depth, or per-base depth to characterize a dataset or support coverage-based SV refinement.

## When NOT to use

- Input is raw FASTQ with no alignment — read QC/length statistics require NanoStat, not coverage computation; a BAM/CRAM is required.
- You need to call SVs or validate SV types other than deletions and duplications — coverage analysis in this paper only supports filtering/polishing of deletions and duplications, and disCoverage is not a variant caller.
- Coverage in the sample is highly non-uniform with local peaks and valleys — coverage alone does not prove an SV, so do not use it as sole evidence for variant detection.

## Inputs

- BAM alignments of nanopore reads (one per aligner: minimap2, NGMLR, lra), e.g. for the NA19240 control dataset against hg38
- Nanopore FASTQ reads for read-length statistics (analyzed with NanoStat, targeting mean read length 14,565 bp)
- MAPQ threshold parameter (20) and optional BED/window arguments for mosdepth

## Outputs

- mosdepth outputs per BAM: {prefix}.mosdepth.global.dist.txt (proportion of bases covered at or above thresholds), {prefix}.mosdepth.summary.txt (mean/min/max depth), optional per-base or threshold BED files
- Genome coverage fraction and median coverage per aligner (e.g. 93/92/91% and 16/17/15× for lra/minimap2/NGMLR)
- Combined summary table (na19240_read_stats_coverage.tsv) with mean read length and per-aligner coverage percentages

## How to apply

Obtain the three BAM alignments (lra, minimap2, NGMLR) of the nanopore reads against the hg38 reference, then run mosdepth separately on each BAM with the mapping-quality flag set to -Q 20 so that reads with MAPQ < 20 are ignored. Use mosdepth's default read filtering (-F 1796) and its distribution output ({prefix}.mosdepth.global.dist.txt) plus the summary file to extract (a) the fraction of the genome covered at or above 1× under the MAPQ ≥ 20 filter and (b) median/mean depth per chromosome and genome-wide. In this study that reproduced 93%/92%/91% genome coverage and median coverage of 16×/17×/15× for lra/minimap2/NGMLR on the NA19240 control, alongside a NanoStat-derived mean read length of 14,565 bp from the FASTQ reads. Compile the per-aligner coverage values with the read-length statistic into a single summary table (e.g. na19240_read_stats_coverage.tsv) so the row can be checked against the published Table 2 within rounding tolerance.

## Related tools

- **mosdepth** (Computes fast BAM/CRAM depth, distribution of bases covered at or above thresholds, and per-chromosome/genome-wide coverage summaries under a MAPQ filter (-Q 20) for each aligner's BAM) — https://github.com/brentp/mosdepth
- **minimap2** (One of the three aligners whose BAM output is the coverage input (NGMLR and lra are the other two))
- **NanoStat** (Derives read-length statistics (mean 14,565 bp) from the NA19240 FASTQ reads to accompany the coverage metrics in the summary table)

## Examples

```
mosdepth -Q 20 --fast-mode NA19240_minimap2 NA19240.minimap2.sorted.bam
```

## Evaluation signals

- Reproduces the NA19240 row of Table 2: mean read length 14,565 bp and genome coverage of 93/92/91% (lra/minimap2/NGMLR) within rounding tolerance
- Median coverage per aligner matches 16×/17×/15× (lra/minimap2/NGMLR)
- The MAPQ filter is actually applied: mosdepth invoked with -Q 20 (reads with quality less than the threshold are ignored), not left at the default of 0
- Per-BAM {prefix}.mosdepth.global.dist.txt contains genome-wide proportion-of-bases-covered lines whose computed covered fraction is consistent with the tabulated percentages
- All three aligner BAMs processed with identical filtering parameters so coverage values are comparable across aligners

## Limitations

- Coverage is not uniform in a sample; local peaks and valleys occur without necessarily indicating an SV, and coverage-based analysis is limited by unresolved genome regions and unclear shift boundaries.
- No software versions were reported for the aligners or analysis tools, so exact reproduction may be sensitive to tool versions.
- Coverage-based filtering applies only to deletions and duplications, not all SV types.

## Evidence

- [abstract] We also used a custom method for the statistical analysis of the coverage value to refine the resulting datasets: "We also used a custom method for the statistical analysis of the coverage value to refine the resulting datasets"
- [methods] (using the great [Mosdepth](https://github.com/brentp/mosdepth) tool): "(using the great [Mosdepth](https://github.com/brentp/mosdepth) tool)"
- [readme] mapping quality threshold. reads with a quality less than this value are ignored [default: 0]: [evidence span withheld — anchor status: external_doc]
- [readme] a distribution of proportion of bases covered at or above a given threshold for each chromosome and genome-wide.: [evidence span withheld — anchor status: external_doc]
- [discussion] Coverage is not uniform in a sample, and local peaks and valleys are frequently observed without necessarily pointing to an SV: "Coverage is not uniform in a sample, and local peaks and valleys are frequently observed without necessarily pointing to an SV"
