---
name: copy-number-variant-region-annotation
description: Use when SV calls from a nanopore variant caller must be turned into
  per-region coverage profiles (with 1-Mb flanks) before disCoverage's coverage-agreement
  statistics or p < 0.05 filtering can be applied.
when_to_use_negative:
- The goal is de novo SV discovery — disCoverage is not a variant caller and needs
  putative SV positions from an additional technique or multi-sample analysis
- Calls include SV types other than deletions and duplications (e.g., insertions,
  inversions) — coverage-based refinement can only be applied to DEL and DUP
- The candidate region is a very large SV (>10 Mb) encompassing other SVs or centromeres
  — such calls were deemed artifacts and not considered true
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: disCoverage
  role: Wraps mosdepth; takes the 4-column SV BED and BAM via -bam/-bed and computes
    per-base coverage of SV intervals with 1-Mb flanks, writing labeled per-region
    coverage outputs
  repo: https://github.com/javiercguard/disCoverage
- name: mosdepth
  role: Per-base and per-region depth calculation invoked internally by disCoverage
    on the SV BED regions, with the MAPQ threshold controlling which reads count
  repo: https://github.com/brentp/mosdepth
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:5289d4f5e0da6190d3522bbfe3eaccc272a281d19d5448099e0f4dcdf5db0a26
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# copy-number-variant-region-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert putative DEL/DUP structural variant calls into a 4-column BED (chromosome, start, end, SV type) and run disCoverage, which internally invokes mosdepth, to obtain per-base coverage over each SV interval plus 1 Mb of up- and downstream flanks. This profile is the prerequisite input for disCoverage's later statistical filtering of nanopore SV calls.

## When to use

Use when SV calls from a nanopore variant caller must be turned into per-region coverage profiles (with 1-Mb flanks) before disCoverage's coverage-agreement statistics or p < 0.05 filtering can be applied.

## When NOT to use

- The goal is de novo SV discovery — disCoverage is not a variant caller and needs putative SV positions from an additional technique or multi-sample analysis
- Calls include SV types other than deletions and duplications (e.g., insertions, inversions) — coverage-based refinement can only be applied to DEL and DUP
- The candidate region is a very large SV (>10 Mb) encompassing other SVs or centromeres — such calls were deemed artifacts and not considered true

## Inputs

- Structural variant calls as a 4-column BED (chromosome, start, end, SV type), restricted to DEL and DUP
- SV calls in VCF format (also accepted by disCoverage)
- Nanopore read alignments in BAM format (e.g., the NA19240 nanopore alignment)
- Output directory path and an analysis label

## Outputs

- Per-sample per-region coverage file named <sample>.<analysisLabel>.per-region.bed.gz (gzip-compressed, tab-delimited region/coverage columns)
- Per-base coverage spanning each SV interval plus 1-Mb up- and downstream flanks, ready for disCoverage's statistical analysis

## How to apply

First convert SV calls into a 4-column BED (chromosome, start, end, SV type), retaining only deletions (DEL) and duplications (DUP) since disCoverage ignores other variant types; VCF-formatted SV calls are also accepted. Then run disCoverage, passing the nanopore alignment BAM (e.g., the NA19240 nanopore alignment) via -bam and the SV BED via -bed; disCoverage internally invokes mosdepth to compute coverage from the BAM, configured with a mapping-quality filter of MAPQ >= 20. Coverage is extracted per base within each SV interval plus 1 Mb of up- and downstream flanking sequence, a margin chosen to tolerate SV coordinate error and local coverage variability. Write results to the output directory (-o) with an analysis label (-i), producing files named <sample>.<analysisLabel>.<etc>, including a per-region coverage bed.gz per sample.

## Related tools

- **disCoverage** (Wraps mosdepth; takes the 4-column SV BED and BAM via -bam/-bed and computes per-base coverage of SV intervals with 1-Mb flanks, writing labeled per-region coverage outputs) — https://github.com/javiercguard/disCoverage
- **mosdepth** (Per-base and per-region depth calculation invoked internally by disCoverage on the SV BED regions, with the MAPQ threshold controlling which reads count) — https://github.com/brentp/mosdepth

## Examples

```
disCoverage -bam NA19240.bam -bed svs.bed -o outdir -i NA19240_analysis
```

## Evaluation signals

- A .per-region.bed.gz coverage file exists for the analysis (e.g., NA19240), is gzip-compressed, and contains the expected tab-delimited region/coverage columns
- Regions in the coverage file span each SV interval plus the 1-Mb up- and downstream flanks
- The input BED is 4-column (chromosome, start, end, SV type) and contains only DEL and DUP rows
- The mosdepth configuration inside disCoverage uses a MAPQ >= 20 mapping-quality filter
- Output files follow the <sample>.<analysisLabel>.<etc> naming pattern in the directory given by -o

## Limitations

- Coverage is not uniform in a sample; local peaks and valleys are frequently observed without necessarily pointing to an SV, and shift boundaries and unresolved genome regions complicate interpretation
- Coverage-based filtering applies only to deletions and duplications, not all SV types
- disCoverage is not a variant caller — putative SV coordinates must come from another technique or multi-sample analysis

## Evidence

- [readme] It takes a list of variants in 4-column bed, or just from VCFs with sructural variants, and checks whether the coverage in the region agrees with the SV type.: "It takes a list of variants in 4-column bed, or just from VCFs with sructural variants, and checks whether the coverage in the region agrees with the SV type."
- [readme] disCoverage is a software tool to refine calling for structural variants using coverage analysis (so it can only check deletions and duplications).: "disCoverage is a software tool to refine calling for structural variants using coverage analysis (so it can only check deletions and duplications)."
- [discussion] disCoverage is not a variant caller and needs putative SV positions, which could be obtained from an additional technique: "disCoverage is not a variant caller and needs putative SV positions, which could be obtained from an additional technique"
- [readme] -Q --mapq <mapq>                  mapping quality threshold. reads with a quality less than this value are ignored [default: 0]: [evidence span withheld — anchor status: external_doc]
