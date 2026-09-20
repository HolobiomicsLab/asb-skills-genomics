---
name: structural-variant-coordinate-bed-encoding
description: Use when SV calls (VCF or coordinate lists from a variant caller or external
  technique) must be reformatted into BED for coverage-based validation with disCoverage/mosdepth,
  retaining only deletions and duplications.
when_to_use_negative:
- The SV set contains types other than deletions and duplications (e.g., insertions,
  inversions, translocations) — disCoverage can only check DEL/DUP
- No putative SV coordinates exist yet — disCoverage is not a variant caller and needs
  positions supplied from another technique or multi-sample analysis
- The goal is de novo SV discovery from coverage alone rather than validation or filtering
  of already-called coordinates
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: disCoverage
  role: Consumes the 4-column SV BED (or SV VCFs), drives mosdepth to obtain per-base
    coverage over SV intervals plus 1-Mb flanks, and produces the per-region coverage
    output used for DEL/DUP filtering statistics
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Performs the actual fast BAM/CRAM depth calculation invoked internally by
    disCoverage, honoring the MAPQ >= 20 threshold and emitting bgzipped per-region
    BED output
  repo: https://github.com/brentp/mosdepth
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:d01f2fb14ac056699db423927dbc8a8fc1fa859700d80747d089552c44b576c8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/javiercguard/disCoverage
  tools:
  - Mosdepth
  - disCoverage
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# structural-variant-coordinate-bed-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes putative SV calls as a 4-column BED (chromosome, start, end, SV type) so disCoverage can invoke mosdepth and compute per-base coverage over each SV interval plus 1-Mb up- and downstream flanks, the prerequisite for its coverage-based statistics.

## When to use

Use when SV calls (VCF or coordinate lists from a variant caller or external technique) must be reformatted into BED for coverage-based validation with disCoverage/mosdepth, retaining only deletions and duplications.

## When NOT to use

- The SV set contains types other than deletions and duplications (e.g., insertions, inversions, translocations) — disCoverage can only check DEL/DUP
- No putative SV coordinates exist yet — disCoverage is not a variant caller and needs positions supplied from another technique or multi-sample analysis
- The goal is de novo SV discovery from coverage alone rather than validation or filtering of already-called coordinates

## Inputs

- Structural variant calls as a VCF or a 4-column BED (chromosome, start, end, SV type), restricted to DEL and DUP
- Nanopore whole-genome alignment BAM (e.g., NA19240 long-read alignments)

## Outputs

- Per-sample per-region coverage file named <sample>.<analysisLabel>.per-region.bed.gz (gzip-compressed, tab-delimited region/coverage columns)
- Coverage profiles spanning each SV interval plus 1-Mb up- and downstream flanks, in the directory given by -o

## How to apply

Convert the SV calls into a 4-column BED with chromosome, start, end, and SV type, keeping only DEL and DUP entries because disCoverage's coverage analysis cannot check other variant types. Run disCoverage on the nanopore alignment (e.g., the NA19240 BAM) with -bam and -bed arguments; disCoverage internally invokes mosdepth, configured with a mapping-quality filter of MAPQ >= 20, to compute per-base coverage within each SV interval plus 1 Mb of flanking sequence on each side. The 1-Mb flanks exist to accommodate SV coordinate error and coverage variability when the downstream statistics are computed. Set the output directory with -o and an analysis label with -i so results are written as <sample>.<analysisLabel>.<etc>, including a per-region coverage bed.gz per sample. Note that disCoverage is not a variant caller: the putative SV positions must come from another technique or multi-sample analysis.

## Related tools

- **disCoverage** (Consumes the 4-column SV BED (or SV VCFs), drives mosdepth to obtain per-base coverage over SV intervals plus 1-Mb flanks, and produces the per-region coverage output used for DEL/DUP filtering statistics) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Performs the actual fast BAM/CRAM depth calculation invoked internally by disCoverage, honoring the MAPQ >= 20 threshold and emitting bgzipped per-region BED output) — https://github.com/brentp/mosdepth

## Examples

```
disCoverage -bam NA19240.bam -bed svs_del_dup.bed -o coverage_out -i NA19240_disCov
```

## Evaluation signals

- A .per-region.bed.gz coverage file exists for the analysis (e.g., NA19240), is gzip-compressed, and contains the expected tab-delimited region/coverage columns
- Regions in the coverage file span each SV interval extended by the 1-Mb up- and downstream flanks
- The input BED has exactly 4 columns with only DEL and DUP values in the 4th (SV type) column, as accepted by disCoverage
- Output files follow the <sample>.<analysisLabel>.<etc> naming convention, with one per-region coverage bed.gz per sample
- mosdepth was run with the MAPQ >= 20 mapping-quality filter so low-quality alignments do not inflate per-base coverage

## Limitations

- Coverage is not uniform in a sample; local peaks and valleys frequently occur without necessarily pointing to an SV, which complicates the downstream statistics
- Coverage-based filtering applies only to deletions and duplications, not all SV types
- SV coordinate discrepancies between reference versions (e.g., 8 SVs whose coordinates could not be translated to hg38) mean some events cannot be encoded or found under a given reference
- The downstream p-value threshold (< 0.05) for coverage support was not confirmed as optimal; further studies are required

## Evidence
