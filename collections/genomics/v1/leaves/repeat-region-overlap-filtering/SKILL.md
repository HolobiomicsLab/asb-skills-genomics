---
name: repeat-region-overlap-filtering
description: Use when a consensus SV call set (e.g. a merged VCF such as the 315-call
  set accepted by at least two callers) must be reduced by dropping calls whose breakpoints
  overlap repeats, low complexity regions, or annotated LTRs before manual inspection.
when_to_use_negative:
- Input is raw per-caller SV predictions before the SURVIVOR merge step has produced
  the consensus set — the merge (at-least-two-callers) step comes first
- The next step is manual vetting by visual inspection of read alignments — this filtering
  only precedes, and does not replace, that curation
- The goal is maximizing sensitivity to detect all SVs — this study explicitly prioritized
  high specificity over high sensitivity, so filtering removes true but unreliable
  calls
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_0157
tools:
- name: SURVIVOR
  role: Provides the 'filter' module to remove SVs from a VCF based on size and/or
    regions to ignore, and 'bincov' to bin coverage to a bed file for filtering SVs
    in low MQ regions; used to reduce the 315-call consensus set before manual inspection
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:060257cd1d7ae2ca5a61f898943da56c20a987c1b2698484dd905bcb297bb92a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Repeat-region-overlap filtering of SV calls

## Summary

Removes structural variant calls whose breakpoints fall in low complexity regions or previously annotated long terminal repeats (LTRs), as applied to the 315-call consensus SV set in the fission yeast S. pombe study prior to manual curation.

## When to use

Use when a consensus SV call set (e.g. a merged VCF such as the 315-call set accepted by at least two callers) must be reduced by dropping calls whose breakpoints overlap repeats, low complexity regions, or annotated LTRs before manual inspection.

## When NOT to use

- Input is raw per-caller SV predictions before the SURVIVOR merge step has produced the consensus set — the merge (at-least-two-callers) step comes first
- The next step is manual vetting by visual inspection of read alignments — this filtering only precedes, and does not replace, that curation
- The goal is maximizing sensitivity to detect all SVs — this study explicitly prioritized high specificity over high sensitivity, so filtering removes true but unreliable calls

## Inputs

- Consensus SV call set VCF (the 315-call set accepted by at least two callers, produced by SURVIVOR merge)
- Annotation of previously annotated long terminal repeats (LTRs) and low complexity regions
- Mapping-quality (MQ) region information (e.g. MQ=0 regions via SURVIVOR bincov bed output)

## Outputs

- Filtered SV VCF (filtered_svs.vcf) with repeat/LTR-overlapping and low-complexity-region calls removed
- Filtered call set for downstream manual inspection of read alignments

## How to apply

Load the consensus SV call set VCF produced by the SURVIVOR merge step (in this study, the 315 calls detected by at least two of Delly, Lumpy, Pindel and cn.MOPs). Run the SURVIVOR 'filter' module, which filters a VCF based on size and/or regions to ignore, to drop low-support calls (paired-end support threshold set to <=10 reads) and calls in MQ=0 regions; the 'bincov' module can bin the coverage vector to a bed file to identify low MQ regions. Intersect SV breakpoints with the annotation of previously annotated long terminal repeats and low complexity regions, and remove any call whose breakpoints overlap them. Write the surviving calls to a filtered VCF (e.g. filtered_svs.vcf) for downstream manual vetting by visual inspection of read alignments. The rationale, per the article, is prioritizing high specificity (correctness of the inferred SV) over high sensitivity, so unreliable repeat-associated calls are discarded before laborious manual curation.

## Related tools

- **SURVIVOR** (Provides the 'filter' module to remove SVs from a VCF based on size and/or regions to ignore, and 'bincov' to bin coverage to a bed file for filtering SVs in low MQ regions; used to reduce the 315-call consensus set before manual inspection) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
./SURVIVOR filter  # see individual parameters with: ./SURVIVOR filter
```

## Evaluation signals

- The filtered VCF exists and parses as valid VCF
- No surviving call has paired-end support <=10 reads
- No surviving SV breakpoint falls in an MQ=0 (zero mapping quality) region
- No surviving SV breakpoint overlaps an annotated LTR or low complexity region
- The filtered set is strictly smaller than the 315-call input consensus set, with only repeat/LTR-overlapping and low-support calls removed

## Limitations

- The study prioritized high specificity over high sensitivity, so not all SVs were detected — repeat-overlapping calls are discarded rather than rescued
- Filtering was applied before manual vetting; surviving calls still required visual inspection of read alignments, so this step alone does not guarantee call correctness
- SURVIVOR version in the README is 1.0.3; contact fritz.sedlazeck@gmail.com for problems with the tool

## Evidence

- [abstract] we further filtered the 315 variants by removing SV calls whose breakpoints overlapped with low complexity regions or any that corresponded to previously annotated long terminal repeats: "we further filtered the 315 variants by removing SV calls whose breakpoints overlapped with low complexity regions or any that corresponded to previously annotated long terminal repeats"
- [discussion] We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs): "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
- [readme] filter	Filter a vcf file based on size and/or regions to ignore: [evidence span withheld — anchor status: external_doc]
- [readme] bincov	Bins coverage vector to a bed file to filter SVs in low MQ regions: [evidence span withheld — anchor status: external_doc]
