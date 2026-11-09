---
name: supplementary-table-cross-referencing
description: Use when a computed call set (e.g. invMap inversion calls on the NA12878
  dataset) must be labeled as ground-truth vs non-ground-truth using the paper's supplementary
  tables/figures, or when published summary statistics must be reproduced.
when_to_use_negative:
- The variant calls are already labeled against ground truth, so supplementary cross-referencing
  would duplicate existing annotations.
- The analysis targets structural-variant types (e.g. deletions, duplications) not
  covered by the paper's supplementary inversion tables S5 and Figures S3-S4.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Produces the inversion calls and alignment identities/anchors that are cross-referenced
    against supplementary tables and competing-mapper call sets
  repo: https://github.com/zhang134/invMap.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:a4532c5ed8a03a0365d3a45ad948738ed7efea94225ef33fca6dd3d10334cc92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - invMap
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# supplementary-table-cross-referencing

## Summary

Cross-references predicted variant/alignment calls against the paper's supplementary tables and figures to label each call as ground-truth-matching or novel, and to recover published summary statistics (e.g. mean identity of non-ground-truth invMap-only inversion calls, 81.95%).

## When to use

Use when a computed call set (e.g. invMap inversion calls on the NA12878 dataset) must be labeled as ground-truth vs non-ground-truth using the paper's supplementary tables/figures, or when published summary statistics must be reproduced.

## When NOT to use

- The variant calls are already labeled against ground truth, so supplementary cross-referencing would duplicate existing annotations.
- The analysis targets structural-variant types (e.g. deletions, duplications) not covered by the paper's supplementary inversion tables S5 and Figures S3-S4.

## Inputs

- invMap inversion-call output (SAM alignments plus chaining anchors) for the NA12878 real human genome sequencing dataset
- Competing-mapper call sets (Winnowmap2, minimap2, NGMLR, lra) for interval-overlap comparison
- Supplementary Table S5 and Figures S3-S4 (ground-truth labels and published statistics)

## Outputs

- novel_inversions_evidence.csv with columns for identity, inversion length, supporting anchors, and truth status
- Summary statistics (e.g. mean identity of non-ground-truth invMap-only calls, 81.95%) and labeled novel-call entries (e.g. the 231 bp inversion)

## How to apply

Extract the predicted calls (e.g. invMap inversion calls with alignment identity, inversion length such as the 231 bp novel inversion, and chaining anchors) and compare them, via interval overlap on reference coordinates, against the call sets from competing mappers (Winnowmap2, minimap2, NGMLR, lra) to isolate invMap-only calls. Then cross-reference each unique call with the paper's Supplementary Table S5 and Figures S3-S4 to label it as matching the simulated ground truth or not, and compute summary statistics over the non-ground-truth calls — here the mean alignment identity, which should equal the reported 81.95%. Validation hinges on reproducing both the known published entries (e.g. the novel 231 bp inversion detected by invMap but missed by all competing mappers) and the published aggregate values; a mismatch indicates incorrect interval overlap logic or a mis-mapped supplementary entry.

## Related tools

- **invMap** (Produces the inversion calls and alignment identities/anchors that are cross-referenced against supplementary tables and competing-mapper call sets) — https://github.com/zhang134/invMap.git

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- The output evidence table reproduces the novel 231 bp inversion entry that is absent from the simulated ground truth.
- The mean alignment identity of non-ground-truth invMap-only calls equals the reported 81.95%.
- No invMap-only novel call overlaps any Winnowmap2, minimap2, NGMLR, or lra call by reference-coordinate interval overlap.
- Every call in the table carries a truth-status label traceable to Supplementary Table S5 or Figures S3-S4.
- Each record includes identity, length, and supporting anchors from the invMap chaining output.

## Limitations

- The 231 bp inversion and 81.95% mean identity apply to the NA12878 real human genome sequencing dataset and the simulated ground truth used in the paper; they are not universal.
- Competing mappers (Winnowmap2, minimap2, NGMLR, lra) report the novel 231 bp inversion as a plain alignment rather than an inversion call, so mapper output format differences can confound interval-overlap comparisons.
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, which constrains reproducibility of the underlying calls.

## Evidence

- [results] For the new 231 bp inversion detected by invMap (which is not part of the simulated ground truth), Winnowmap2, minimap2, NGMLR, and lra fail to discover the new inversion: [evidence span withheld — anchor status: not_found]
- [methods] Cross-reference the extracted calls with Supplementary Table S5 and Figures S3-S4 to label ground-truth-matching vs non-ground-truth calls and compute the mean identity of the latter (81.95%).: [evidence span withheld — anchor status: not_found]
- [abstract] The real human genome sequencing dataset of NA12878 illustrates that invMap can effectively find more candidate variant calls for inversions than the competing methods: "The real human genome sequencing dataset of NA12878 illustrates that invMap can effectively find more candidate variant calls for inversions than the competing methods"
- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] The final mapping result is invMap_aligned.sam in SAM format.: "The final mapping result is invMap_aligned.sam in SAM format."
