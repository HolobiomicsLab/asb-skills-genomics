---
name: inversion-variant-call-comparison-across-mappers
description: Use when a long-read mapping study yields inversion SV call sets from
  multiple mappers and you need to identify calls unique to invMap, distinguishing
  ground-truth-matching from non-ground-truth novel calls.
when_to_use_negative:
- Input consists only of single-mapper SV calls with no competing call sets, so no
  cross-mapper uniqueness comparison is possible
- The variants of interest are deletions, duplications, or translocations rather than
  inversions, which invMap's remaining-anchor check is designed to discover
- The dataset is short-read sequencing, since invMap targets long noisy reads from
  PacBio or Oxford Nanopore sequencers
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Primary mapper whose specifically designed scoring method for chaining and
    remaining-anchor check produces the inversion calls being isolated and characterized
  repo: https://github.com/zhang134/invMap.git
- name: Winnowmap2
  role: Competing long-read mapper whose call set is compared against invMap calls
    to confirm uniqueness of novel inversion calls
- name: minimap2
  role: Competing long-read mapper whose call set is compared against invMap calls
    to confirm uniqueness of novel inversion calls
- name: NGMLR
  role: Competing long-read mapper whose call set is compared against invMap calls
    to confirm uniqueness of novel inversion calls
- name: lra
  role: Competing long-read mapper whose call set is compared against invMap calls
    to confirm uniqueness of novel inversion calls
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:acc47716d60a5d2f0a1f4cb85ce63dbbec1b19f444cd4e1ba223888f66706650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# inversion-variant-call-comparison-across-mappers

## Summary

Compare inversion SV calls from invMap against competing long-read mappers (Winnowmap2, minimap2, NGMLR, lra) on the NA12878 real human genome sequencing dataset to isolate invMap-only inversion calls, such as a novel 231 bp inversion absent from the simulated ground truth.

## When to use

Use when a long-read mapping study yields inversion SV call sets from multiple mappers and you need to identify calls unique to invMap, distinguishing ground-truth-matching from non-ground-truth novel calls.

## When NOT to use

- Input consists only of single-mapper SV calls with no competing call sets, so no cross-mapper uniqueness comparison is possible
- The variants of interest are deletions, duplications, or translocations rather than inversions, which invMap's remaining-anchor check is designed to discover
- The dataset is short-read sequencing, since invMap targets long noisy reads from PacBio or Oxford Nanopore sequencers

## Inputs

- invMap inversion-call output (SAM alignment / chaining anchors) for the NA12878 real human genome sequencing dataset
- Competing-mapper inversion call sets from Winnowmap2, minimap2, NGMLR, and lra
- Simulated ground-truth inversion annotations and Supplementary Table S5 / Figures S3-S4
- Reference genome coordinate intervals for call-set overlap

## Outputs

- novel_inversions_evidence.csv with columns for alignment identity, inversion length, supporting anchors, and ground-truth status
- List of invMap-only inversion calls (e.g. the 231 bp novel inversion) with mean identity of non-ground-truth calls (81.95%)

## How to apply

Load the invMap inversion-call outputs for the NA12878 real human genome sequencing dataset together with the competing mappers' call sets (Winnowmap2, minimap2, NGMLR, lra). Compare invMap calls against the other mappers' calls via interval overlap on the reference coordinates to isolate inversions predicted only by invMap. For each unique invMap call, record alignment identity, inversion length (e.g. the 231 bp novel inversion), and the supporting anchors from the invMap chaining output. Cross-reference the extracted calls with Supplementary Table S5 and Figures S3-S4 to label ground-truth-matching versus non-ground-truth calls, and compute the mean identity of the latter (81.95%). Write the assembled records to a CSV such as novel_inversions_evidence.csv with columns for identity, length, anchors, and truth status.

## Related tools

- **invMap** (Primary mapper whose specifically designed scoring method for chaining and remaining-anchor check produces the inversion calls being isolated and characterized) — https://github.com/zhang134/invMap.git
- **Winnowmap2** (Competing long-read mapper whose call set is compared against invMap calls to confirm uniqueness of novel inversion calls)
- **minimap2** (Competing long-read mapper whose call set is compared against invMap calls to confirm uniqueness of novel inversion calls)
- **NGMLR** (Competing long-read mapper whose call set is compared against invMap calls to confirm uniqueness of novel inversion calls)
- **lra** (Competing long-read mapper whose call set is compared against invMap calls to confirm uniqueness of novel inversion calls)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- The evidence table reproduces the novel 231 bp inversion entry that is not part of the simulated ground truth
- The mean identity of non-ground-truth invMap-only calls equals the reported 81.95%
- For the novel 231 bp inversion, Winnowmap2, minimap2, NGMLR, and lra all fail to discover the inversion and instead report the alignment
- invMap-only calls are confirmed by interval overlap on reference coordinates showing no overlap with competing-mapper call sets
- Each record in novel_inversions_evidence.csv carries non-empty identity, length, anchors, and truth-status fields

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Novel invMap-only calls are not part of the simulated ground truth, so their validity rests on evidence-table curation rather than truth matching
- invMap 1.0.0 (2023/09/08) is the documented version; no changelog is available for behavioral changes

## Evidence

- [abstract] invMap-only call on NA12878: "The real human genome sequencing dataset of NA12878 illustrates that invMap can effectively find more candidate variant calls for inversions than the competing methods"
- [abstract] chaining then inversion discovery: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [results] novel 231 bp inversion missed by baselines: [evidence span withheld — anchor status: not_found]
- [intro] RAM requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] SAM output usage: "The final mapping result is invMap_aligned.sam in SAM format."
