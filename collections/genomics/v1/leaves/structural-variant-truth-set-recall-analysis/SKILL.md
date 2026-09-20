---
name: structural-variant-truth-set-recall-analysis
description: Use when a reproduced nanopore variant-calling pipeline (VCFs from Sniffles/CuteSV/SVIM/NanoVar
  on minimap2/NGMLR/lra alignments) must be benchmarked against a published gold-standard
  SV set, split by SV type and size.
when_to_use_negative:
- The SVs of interest are not deletions or duplications — disCoverage can only check
  DEL/DUP, so coverage-support against a truth set cannot be applied to other SV types
- No established truth set or external reference variant set exists for the sample,
  so recall cannot be computed and only internal filtering (e.g., aCGH comparison
  in patients) is possible
- A standalone variant caller is needed — disCoverage is not a variant caller and
  requires putative SV positions supplied as input
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0625
tools:
- name: disCoverage
  role: Converts VCF SV call sets to 4-column BED and checks coverage support for
    truth-set deletions and duplications using p < 0.05 plus coverage difference
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Computes per-region coverage depth for the NA19240 BAM that disCoverage consumes;
    existing Mosdepth output can be reused with --skip-mosdepth
  repo: https://github.com/brentp/mosdepth
- name: minimap2
  role: One of the aligners whose NA19240 SV calls (alongside lra and NGMLR) are benchmarked
    for truth-set recall
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:f1a0f1bdcfd8daebf2faf90fb62411da6c969dd33691b1147aecd055e7ce02c6
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
  - minimap2
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# structural-variant-truth-set-recall-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how much of an external SV/CNV truth set (e.g., the Chaisson et al. 2019 high-confidence set for NA19240) is recovered by a nanopore SV-calling pipeline, using disCoverage coverage-support checks and per-aligner, per-SV-type recall metrics.

## When to use

Use when a reproduced nanopore variant-calling pipeline (VCFs from Sniffles/CuteSV/SVIM/NanoVar on minimap2/NGMLR/lra alignments) must be benchmarked against a published gold-standard SV set, split by SV type and size.

## When NOT to use

- The SVs of interest are not deletions or duplications — disCoverage can only check DEL/DUP, so coverage-support against a truth set cannot be applied to other SV types
- No established truth set or external reference variant set exists for the sample, so recall cannot be computed and only internal filtering (e.g., aCGH comparison in patients) is possible
- A standalone variant caller is needed — disCoverage is not a variant caller and requires putative SV positions supplied as input

## Inputs

- Merged NA19240 SV call set VCFs produced by the pipeline for each aligner/variant-caller combination
- Chaisson et al. 2019 high-confidence truth-set SVs for NA19240
- NA19240 nanopore BAM alignment file (produced by minimap2, NGMLR, or lra)
- Existing Mosdepth per-region coverage files for NA19240

## Outputs

- 4-column BED files (chr, start, end, SV type) of truth-set and called SVs
- Per-region Mosdepth coverage for the NA19240 BAM
- Coverage-support flags (p < 0.05 plus observed coverage difference) for each truth-set DEL/DUP
- Single TSV table of recall per aligner, split by SV type and size class, including >50-kb agreement counts and median coordinate differences

## How to apply

Load the merged NA19240 SV call sets (VCF) for each aligner/variant-caller combination and the Chaisson et al. 2019 truth-set SVs, then convert both to the 4-column BED format (chr, start, end, SV type) that disCoverage accepts, keeping only deletions and duplications because disCoverage only checks DEL/DUP. Run disCoverage on the NA19240 BAM with Mosdepth coverage to obtain per-region coverage, marking a truth-set event as supported using disCoverage's criterion of p-value < 0.05 combined with the observed coverage difference. Compute recall per aligner (lra/minimap2/NGMLR), split by SV type and size class, including the >50-kb CNV agreement count (0/1/0 SVs out of 122), and the median coordinate differences between matched truth-set and called SVs (~12.5/13.0/14.0 bp). Write all metrics to a single TSV table split by SV type and size for comparison against the paper's reported values.

## Related tools

- **disCoverage** (Converts VCF SV call sets to 4-column BED and checks coverage support for truth-set deletions and duplications using p < 0.05 plus coverage difference) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Computes per-region coverage depth for the NA19240 BAM that disCoverage consumes; existing Mosdepth output can be reused with --skip-mosdepth) — https://github.com/brentp/mosdepth
- **minimap2** (One of the aligners whose NA19240 SV calls (alongside lra and NGMLR) are benchmarked for truth-set recall)

## Examples

```
disCoverage --skip-mosdepth -cov NA19240.mosdepth.regions.bed.gz chaisson_truthset.bed
```

## Evaluation signals

- Computed deletion recall against the Chaisson truth set falls in 37–43% (expected 37.93%/42.71%/40.05% for lra/minimap2/NGMLR)
- Computed duplication recall falls in 3–17% (expected 3.70%/13.81%/16.43%)
- The >50-kb CNV agreement count matches 0/1/0 SVs out of 122 (lra/minimap2/NGMLR)
- Median coordinate differences between matched truth-set and called SVs are ~12.5/13.0/14.0 bp, within rounding tolerance of the paper
- All truth-set events marked supported satisfy the p < 0.05 threshold combined with the observed coverage difference

## Limitations

- disCoverage can only be applied to deletions and duplications, not all SV types
- Coverage analysis is limited by non-uniform coverage, local peaks and valleys, unresolved genome regions, and unclear shift boundaries
- The p-value threshold of 0.05 was not confirmed as optimal; further studies are required to confirm its optimality
- disCoverage is not a variant caller and needs putative SV positions, which could be obtained from an additional technique

## Evidence

- [discussion] Truth-set support comparison: [evidence span withheld — anchor status: not_found]
