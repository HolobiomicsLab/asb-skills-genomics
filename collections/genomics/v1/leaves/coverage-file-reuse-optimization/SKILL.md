---
name: coverage-file-reuse-optimization
description: Use when you already have putative SV coordinates (e.g. from Sniffles/CuteSV/SVIM/NanoVar
  on NGMLR-aligned nanopore reads) and need per-base coverage over each SV interval
  plus 1-Mb flanks to judge coverage support or filter calls.
when_to_use_negative:
- The variant list contains SV types other than deletions and duplications (inversions,
  insertions, translocations) — coverage-based checking can only be applied to DEL/DUP.
- No putative SV positions exist yet — disCoverage is not a variant caller and needs
  coordinates obtained from a variant caller or another technique first.
- You need de novo CNV discovery from raw reads — this skill only evaluates coverage
  agreement at pre-specified candidate regions.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: disCoverage
  role: 'Wraps the analysis: takes the 4-column BED of DEL/DUP calls and the nanopore
    BAM, invokes mosdepth, and writes per-region coverage outputs used for coverage-support
    filtering'
  repo: https://github.com/javiercguard/disCoverage
- name: mosdepth
  role: Performs the fast BAM/CRAM per-base depth calculation with MAPQ >= 20 within
    the SV intervals and their 1-Mb flanks
  repo: https://github.com/brentp/mosdepth
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:232722d079182333d5a9dd7260839886e73d0420b3191e91195ad3c0de794ed8
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
  - disCoverage
  - mosdepth
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# coverage-file-reuse-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives per-base coverage profiles for putative DEL/DUP SV calls and their 1-Mb flanking regions by running disCoverage, which internally invokes mosdepth on a nanopore alignment BAM, and reuses the resulting per-region coverage bed.gz files for downstream coverage-support filtering instead of recomputing depth.

## When to use

Use when you already have putative SV coordinates (e.g. from Sniffles/CuteSV/SVIM/NanoVar on NGMLR-aligned nanopore reads) and need per-base coverage over each SV interval plus 1-Mb flanks to judge coverage support or filter calls.

## When NOT to use

- The variant list contains SV types other than deletions and duplications (inversions, insertions, translocations) — coverage-based checking can only be applied to DEL/DUP.
- No putative SV positions exist yet — disCoverage is not a variant caller and needs coordinates obtained from a variant caller or another technique first.
- You need de novo CNV discovery from raw reads — this skill only evaluates coverage agreement at pre-specified candidate regions.

## Inputs

- Nanopore whole-genome alignment BAM (e.g. NA19240 nanopore reads)
- Structural variant calls as a 4-column BED (chromosome, start, end, SV type) or a VCF containing SVs
- Output directory path and analysis label

## Outputs

- <sample>.<analysisLabel>.per-region.bed.gz per-sample coverage file (gzip-compressed, tab-delimited region/coverage columns)
- Per-base coverage profiles spanning each SV interval plus 1 Mb of up- and downstream flanking sequence

## How to apply

Convert SV calls into a 4-column BED (chromosome, start, end, SV type), retaining only deletions (DEL) and duplications (DUP) because disCoverage can only check these types via coverage analysis. Run disCoverage with the nanopore alignment BAM (-bam, e.g. NA19240) and the SV BED file (-bed); it internally invokes mosdepth with a mapping-quality filter of MAPQ >= 20 to compute per-base coverage. Configure the analysis to cover each SV interval plus 1 Mb of up- and downstream flanking sequence, which accounts for coordinate error and coverage variability (e.g. hg38-vs-hg19 coordinate discrepancies). Write results to an output directory (-o) with an analysis label (-i), producing files named <sample>.<analysisLabel>.<etc>, including a per-region coverage bed.gz per sample. Reuse these bed.gz coverage files for later filtering steps (e.g. p-value < 0.05 combined with coverage difference) rather than re-running depth calculation.

## Related tools

- **disCoverage** (Wraps the analysis: takes the 4-column BED of DEL/DUP calls and the nanopore BAM, invokes mosdepth, and writes per-region coverage outputs used for coverage-support filtering) — https://github.com/javiercguard/disCoverage
- **mosdepth** (Performs the fast BAM/CRAM per-base depth calculation with MAPQ >= 20 within the SV intervals and their 1-Mb flanks) — https://github.com/brentp/mosdepth

## Examples

```
disCoverage -bam NA19240.bam -bed sv_calls.bed -o results/ -i NA19240
```

## Evaluation signals

- A .per-region.bed.gz coverage file exists for the NA19240 analysis, is gzip-compressed, and contains the expected tab-delimited region/coverage columns.
- Regions in the per-region coverage file span each SV interval with the 1-Mb up- and downstream flanks, not just the call boundaries.
- The input BED has exactly 4 columns (chromosome, start, end, SV type) and contains only DEL and DUP entries.
- Output files follow the <sample>.<analysisLabel>.<etc> naming convention in the directory passed via -o.
- The mosdepth invocation applies the MAPQ >= 20 mapping-quality filter (reads with quality below the threshold are ignored).

## Limitations

- Coverage is not uniform in a sample; local peaks and valleys are frequently observed without necessarily pointing to an SV, and shift boundaries can be unclear.
- Coverage-based filtering applies only to deletions and duplications, not to all SV types.
- disCoverage is not a variant caller: it requires putative SV positions supplied from variant calling or an additional technique.
- The p < 0.05 filtering threshold combined with coverage difference was not confirmed as optimal; further studies are required.

## Evidence
