---
name: per-base-coverage-computation-from-bam
description: Use when a BAM/CRAM (e.g., NA19240 nanopore reads aligned with NGMLR/minimap2/lra)
  must be converted into per-base or per-region coverage over SV intervals plus 1-Mb
  flanks for coverage-support analysis.
when_to_use_negative:
- The SVs to assess are not deletions or duplications — the article states coverage-based
  checking 'can only be applied to some types, namely deletions and duplications'.
- Per-base or per-region coverage already exists (e.g., a prior mosdepth run or d4
  file) — recomputation from BAM is redundant.
- Only quantized/thresholded coverage labels are needed; use mosdepth --quantize or
  --thresholds instead of per-base output, which is slower.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mosdepth
  role: Computes per-base and per-region depth from the BAM/CRAM; runs on SV coordinates
    extended by 1-Mb flanks to supply coverage distributions for support testing
  repo: https://github.com/brentp/mosdepth
- name: disCoverage
  role: Consumes the coverage over the 4-column BED SV intervals and surroundings
    to label each call supported/discarded via a >=40% shift and Welch t-test p <
    0.05
  repo: https://github.com/javiercguard/disCoverage
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:6f15ad96b3701796e8e4a0eed703bb44be86e20ab68d5c62ce89255e8d66cece
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
  - disCoverage
  - mosdepth
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-base-coverage-computation-from-bam

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-base and per-region sequencing depth from nanopore BAM/CRAM alignments using mosdepth, providing the coverage distributions that disCoverage compares (Welch t-test, >=40% shift, p<0.05) to decide whether putative DEL/DUP structural variants are supported.

## When to use

Use when a BAM/CRAM (e.g., NA19240 nanopore reads aligned with NGMLR/minimap2/lra) must be converted into per-base or per-region coverage over SV intervals plus 1-Mb flanks for coverage-support analysis.

## When NOT to use

- The SVs to assess are not deletions or duplications — the article states coverage-based checking 'can only be applied to some types, namely deletions and duplications'.
- Per-base or per-region coverage already exists (e.g., a prior mosdepth run or d4 file) — recomputation from BAM is redundant.
- Only quantized/thresholded coverage labels are needed; use mosdepth --quantize or --thresholds instead of per-base output, which is slower.

## Inputs

- BAM or CRAM alignment file (nanopore whole-genome reads, e.g., NA19240)
- Reference FASTA (required for CRAM via --fasta)
- 4-column BED of putative SVs (DEL/DUP entries) optionally extended by 1-Mb flanking regions
- Optional BED file of regions or integer window size for --by

## Outputs

- Gzipped per-base depth BED ({prefix}.per-base.bed.gz) with CSI index
- Mean per-region depth ({prefix}.regions.bed.gz) when --by is given
- Coverage summary ({prefix}.mosdepth.summary.txt) and distribution files
- Coverage values feeding disCoverage's Welch t-test support/discard labels

## How to apply

Run mosdepth on the alignment file to emit per-base depth (default .per-base.bed.gz output) and, when a region BED is supplied, mean depth per region. For disCoverage-style support checks, first build a BED of putative DEL/DUP coordinates (4-column BED from the variant callers) extended by 1-Mb flanking windows on both sides, then use those regions so the coverage inside the SV interval and its surroundings can be compared. Compute the relative coverage shift between the SV interval and the flanking windows and test significance with a Welch two-tailed Student's t-test in R; label a call supported when the shift is >=40% and p < 0.05. mosdepth is chosen because it computes per-base depth roughly twice as fast as samtools depth, making genome-wide extraction over many SV intervals practical.

## Related tools

- **mosdepth** (Computes per-base and per-region depth from the BAM/CRAM; runs on SV coordinates extended by 1-Mb flanks to supply coverage distributions for support testing) — https://github.com/brentp/mosdepth
- **disCoverage** (Consumes the coverage over the 4-column BED SV intervals and surroundings to label each call supported/discarded via a >=40% shift and Welch t-test p < 0.05) — https://github.com/javiercguard/disCoverage

## Examples

```
mosdepth NA19240.disCov NA19240.bam && disCoverage --bed svs_del_dup.bed --coverage NA19240.disCov.per-base.bed.gz
```

## Evaluation signals

- {prefix}.per-base.bed.gz and {prefix}.mosdepth.summary.txt exist, are bgzipped/indexed where applicable, and cover every queried SV region plus flanks.
- Every SV labelled supported by disCoverage satisfies both the >=40% relative coverage shift and Welch t-test p < 0.05 criteria.
- The support/discard table contains exactly one labelled row per input DEL/DUP SV from the 4-column BED.
- Per-base depth values fall within the genome-wide coverage distribution in {prefix}.mosdepth.global.dist.txt for the sample (e.g., ~30X WGS scale).
- DEL intervals show lower mean coverage than their flanking windows and DUP intervals higher, consistent with the called SV type.

## Limitations

- Coverage is not uniform; local peaks and valleys occur without pointing to an SV, so shifts near the 40% boundary can be misleading.
- Coverage-based support applies only to deletions and duplications, not to other SV types.
- The p < 0.05 threshold was not confirmed as optimal; further studies are required to confirm its optimality.
- Unresolved genome regions and unclear shift boundaries limit interpretation of flanking-window comparisons.

## Evidence

- [readme] per-base depth about 2x as fast `samtools depth`--about 25 minutes of CPU time for a 30X genome.: [evidence span withheld — anchor status: external_doc]
