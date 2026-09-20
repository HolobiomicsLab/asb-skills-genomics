---
name: cnv-call-filtering-by-read-depth
description: Use when you have a VCF/BED of >50-kb DEL/DUP calls from a nanopore pipeline
  plus a BAM alignment, and need to remove calls lacking coverage evidence or verify
  coverage support for truth-set SVs.
when_to_use_negative:
- SV types other than deletions and duplications — coverage-based filtering cannot
  be applied to insertions, inversions, or translocations
- You need a de novo variant caller — disCoverage is not a variant caller and requires
  putative SV positions from another technique or multi-sample analysis
- Coverage is highly non-uniform with strong local peaks/valleys or unresolved genome
  regions, which can produce spurious coverage differences
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: disCoverage
  role: 'Core tool: runs the statistical coverage-difference test on each SV and filters
    calls lacking coverage support'
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Invoked by disCoverage to compute per-region mean coverage over each queried
    SV interval
  repo: https://github.com/brentp/mosdepth
- name: minimap2
  role: Upstream aligner producing the nanopore BAM and one of the SV call sets being
    filtered
- name: NGMLR
  role: Upstream aligner producing one of the nanopore SV call sets being filtered
- name: lra
  role: Upstream aligner producing one of the nanopore SV call sets being filtered
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:0e65c8500651f02c91e9c801353fde97d1f43fd6bdc38bc0d6abd79d5ec593ad
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
  - NGMLR
  - disCoverage
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cnv-call-filtering-by-read-depth

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refine nanopore structural variant calls by statistically testing read-depth coverage over each candidate deletion/duplication with disCoverage, discarding calls whose coverage does not support the claimed copy-number change. This addresses the poor specificity of variant callers for large (>50 kb) CNVs in nanopore WGS data.

## When to use

Use when you have a VCF/BED of >50-kb DEL/DUP calls from a nanopore pipeline plus a BAM alignment, and need to remove calls lacking coverage evidence or verify coverage support for truth-set SVs.

## When NOT to use

- SV types other than deletions and duplications — coverage-based filtering cannot be applied to insertions, inversions, or translocations
- You need a de novo variant caller — disCoverage is not a variant caller and requires putative SV positions from another technique or multi-sample analysis
- Coverage is highly non-uniform with strong local peaks/valleys or unresolved genome regions, which can produce spurious coverage differences

## Inputs

- Merged >50-kb DEL/DUP call sets from nanopore pipeline in VCF format (e.g., from lra, minimap2, or NGMLR alignment)
- 4-column BED file of SVs (chr, start, end, SVTYPE) — disCoverage can generate this directly from .vcf/.vcf.gz
- Nanopore WGS BAM alignment file (e.g., NA19240 PromethION data)
- Gold-standard truth-set SVs (e.g., Chaisson et al. NA19240 high-confidence set) for validation
- Optional: cached per-region coverage files (<P1.label.per-region.bed.gz>) for --skip-mosdepth reruns

## Outputs

- Filtered SV call set with retained/discarded status and coverage-support flags per call
- Per-region coverage files (<P1.label.per-region.bed.gz>) from Mosdepth
- Counts of discarded false-positive calls per aligner and coverage-support status of truth-set SVs

## How to apply

Convert the merged SV call sets (VCF or .vcf.gz) and any reference/truth SVs into 4-column BED (chr, start, end, SVTYPE); disCoverage accepts VCF inputs directly. Run disCoverage once per call set with the nanopore BAM (-bam), the SV BED (-bed), an output directory (-o), an analysis label (-i), and at least 4 threads (-t); this invokes Mosdepth to compute per-region coverage for every queried SV. Apply disCoverage's statistical coverage-difference test with a p-value threshold smaller than 0.05 to flag each deletion/duplication as coverage-supported or not (non-DEL/DUP types are ignored). Discard calls lacking coverage support and record retained/discarded status with support flags per call. For repeat runs over multiple aligners' call sets, reuse cached coverage by passing the prior <P1.label.per-region.bed.gz> files via -cov together with --skip-mosdepth. The rationale is that a true deletion or duplication must produce a coverage drop or shift, so variant-caller calls without such coverage evidence are likely false positives.

## Related tools

- **disCoverage** (Core tool: runs the statistical coverage-difference test on each SV and filters calls lacking coverage support) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Invoked by disCoverage to compute per-region mean coverage over each queried SV interval) — https://github.com/brentp/mosdepth
- **minimap2** (Upstream aligner producing the nanopore BAM and one of the SV call sets being filtered)
- **NGMLR** (Upstream aligner producing one of the nanopore SV call sets being filtered)
- **lra** (Upstream aligner producing one of the nanopore SV call sets being filtered)

## Examples

```
disCoverage -bam NA19240.bam -bed calls_50kb.bed -o out/ -i NA19240_minimap2 -t 4
```

## Evaluation signals

- Discarded-call counts reproduce the reported 83/115/114 (lra/minimap2/NGMLR) SVs absent from the gold standard
- Specificity increase for minimap2 and NGMLR call sets is approximately 56%/62%
- Coverage-support status is assigned to 55/61 of the 122 Chaisson et al. truth-set SVs
- Reruns with --skip-mosdepth and -cov yield identical filtering results to fresh Mosdepth runs
- No non-DEL/DUP SV types appear in the filtering output

## Limitations

- Coverage-based filtering only applies to deletions and duplications, not all SV types
- The p-value threshold of 0.05 was not confirmed as optimal; further studies are required
- Coverage is not uniform in a sample; local peaks and valleys are frequently observed without necessarily pointing to an SV, and unresolved genome regions or unclear shift boundaries limit accuracy
- disCoverage is not a variant caller and needs putative SV positions from an additional technique

## Evidence

- [readme] disCoverage is a software tool to refine calling for structural variants using coverage analysis (so it can only check deletions and duplications).: "disCoverage is a software tool to refine calling for structural variants using coverage analysis (so it can only check deletions and duplications)."
- [discussion] p-value threshold < 0.05: "we used a threshold for p-values smaller than 0.05"
- [results] discarded 83, 115, and 114 SVs: [evidence span withheld — anchor status: not_found]
- [discussion] disCoverage not a variant caller: "disCoverage is not a variant caller and needs putative SV positions, which could be obtained from an additional technique"
- [readme] mosdepth per-region output: [evidence span withheld — anchor status: not_found]
