---
name: nanopore-bam-coverage-profiling
description: Use when putative DEL/DUP SV calls (from Sniffles, CuteSV, SVIM, NanoVar
  on nanopore WGS alignments) must be profiled by per-base coverage across each SV
  interval plus 1-Mb up- and downstream flanks before coverage-based filtering or
  truth-set validation.
when_to_use_negative:
- The SV calls are insertions, inversions, or other non-DEL/DUP types — coverage analysis
  'can only be applied to some types, namely deletions and duplications'
- No putative SV coordinates exist — disCoverage is not a variant caller and needs
  positions from another technique or multi-sample analysis
- The candidate SV is very large (>10 Mb) and encompasses other SVs or centromeres
  — such calls were treated as probable artifacts, not true variants
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: disCoverage
  role: Converts the 4-column DEL/DUP BED (or SV VCF) into coverage analysis regions
    with 1-Mb flanks and invokes mosdepth internally against the nanopore BAM
  repo: https://github.com/javiercguard/disCoverage
- name: mosdepth
  role: Computes the per-base depth from the BAM under the MAPQ >= 20 mapping-quality
    filter within each SV-plus-flank interval
  repo: https://github.com/brentp/mosdepth
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:ce5dbd331d72c58ddbfa56848f9c2f6300af17c7b95a730fdee7b4dfce145dee
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nanopore-bam-coverage-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles per-base read depth from nanopore alignment BAMs over putative DEL/DUP structural variants and their 1-Mb flanking sequence, using disCoverage which internally invokes mosdepth. This coverage profile is the raw material disCoverage needs to statistically test whether called SVs agree with observed DNA copy number.

## When to use

Use when putative DEL/DUP SV calls (from Sniffles, CuteSV, SVIM, NanoVar on nanopore WGS alignments) must be profiled by per-base coverage across each SV interval plus 1-Mb up- and downstream flanks before coverage-based filtering or truth-set validation.

## When NOT to use

- The SV calls are insertions, inversions, or other non-DEL/DUP types — coverage analysis 'can only be applied to some types, namely deletions and duplications'
- No putative SV coordinates exist — disCoverage is not a variant caller and needs positions from another technique or multi-sample analysis
- The candidate SV is very large (>10 Mb) and encompasses other SVs or centromeres — such calls were treated as probable artifacts, not true variants

## Inputs

- Nanopore whole-genome alignment BAM (e.g., the NA19240 nanopore BAM)
- 4-column BED of putative SV calls (chromosome, start, end, SV type) restricted to DEL and DUP records
- Optional VCF of structural variants, convertible to the 4-column BED

## Outputs

- Per-region coverage bed.gz file per sample (e.g., <sample>.<analysisLabel>.per-region.bed.gz), gzip-compressed and tab-delimited with region and coverage columns
- Coverage profile spanning each SV interval extended by 1 Mb of up- and downstream flanking sequence

## How to apply

First convert the SV calls to a 4-column BED (chromosome, start, end, SV type), keeping only deletions (DEL) and duplications (DUP) since disCoverage's coverage analysis can only check those types. Then run disCoverage with the nanopore alignment BAM (-bam), the SV BED file (-bed), an output directory (-o), and an analysis label (-i); disCoverage internally invokes mosdepth to compute per-base coverage from the BAM. Configure the coverage step with a mapping-quality filter of MAPQ >= 20 so poorly aligned reads do not distort the depth signal. Regions are extended 1 Mb in each direction beyond the SV boundaries to absorb coordinate error and local coverage variability. Results are written as <sample>.<analysisLabel>.<etc> files in the output directory, including a per-region coverage bed.gz per sample that feeds the downstream coverage statistics (p < 0.05 threshold) used for polishing calls.

## Related tools

- **disCoverage** (Converts the 4-column DEL/DUP BED (or SV VCF) into coverage analysis regions with 1-Mb flanks and invokes mosdepth internally against the nanopore BAM) — https://github.com/javiercguard/disCoverage
- **mosdepth** (Computes the per-base depth from the BAM under the MAPQ >= 20 mapping-quality filter within each SV-plus-flank interval) — https://github.com/brentp/mosdepth

## Examples

```
disCoverage -bam NA19240.bam -bed svs.bed -o results -i NA19240
```

## Evaluation signals

- A .per-region.bed.gz coverage file exists for the analysis (e.g., NA19240), is gzip-compressed, and is tab-delimited with the expected region/coverage columns
- Every reported coverage region spans the corresponding SV interval extended by 1 Mb of upstream and downstream sequence
- The input BED is 4-column (chromosome, start, end, SV type) and contains only DEL and DUP records; other variant types were excluded
- The mosdepth invocation used a MAPQ >= 20 threshold (reads with quality below this value are ignored)
- Output filenames follow the <sample>.<analysisLabel>.<etc> pattern in the directory passed with -o

## Limitations

- Coverage is not uniform in a sample; local peaks and valleys are frequently observed without necessarily pointing to an SV, so profiles must be interpreted statistically
- The coverage-support p-value threshold (< 0.05) has not been confirmed as optimal and requires further study
- Coverage-based analysis applies only to deletions and duplications, not all SV types, and disCoverage needs putative SV positions supplied from another technique
- Unresolved genome regions and unclear shift boundaries limit the reliability of coverage comparisons

## Evidence
