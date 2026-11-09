---
name: genomic-interval-overlap-analysis
description: Use when you have two or more structural-variant call sets in reference
  coordinates and need to isolate tool-specific calls (e.g., inversions found by invMap
  but missed by competing mappers).
when_to_use_negative:
- The call sets are already deduplicated and cross-mapped between mappers, so overlap
  analysis would be redundant.
- The variant type is not an inversion detectable by long-read mapping (e.g., small
  indels or SNPs), which is outside invMap's inversion-focused scope.
- You only have a single call set with no competing mapper calls to compare against.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: produces the inversion-call set (SAM alignment output with anchors) that is
    compared against competing mappers via interval overlap
  repo: https://github.com/zhang134/invMap.git
- name: PacBio
  role: long-read sequencing source whose reads span inversion breakpoints, generating
    the call sets compared by interval overlap
- name: Oxford Nanopore
  role: long-read sequencing source whose reads span inversion breakpoints, generating
    the call sets compared by interval overlap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:f89cd99f9d8e12dc0bb2ee3ec4bc956f4d41e989d17245ea458626ec95c72145
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - Oxford Nanopore
  - PacBio
  - invMap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# genomic-interval-overlap-analysis

## Summary

Compares inversion/SV call sets from long-read mappers by interval overlap on reference coordinates to isolate calls unique to one tool. Applied to invMap vs Winnowmap2, minimap2, NGMLR, and lra on NA12878 real human genome sequencing data to surface invMap-only inversion calls such as the novel 231 bp inversion.

## When to use

Use when you have two or more structural-variant call sets in reference coordinates and need to isolate tool-specific calls (e.g., inversions found by invMap but missed by competing mappers).

## When NOT to use

- The call sets are already deduplicated and cross-mapped between mappers, so overlap analysis would be redundant.
- The variant type is not an inversion detectable by long-read mapping (e.g., small indels or SNPs), which is outside invMap's inversion-focused scope.
- You only have a single call set with no competing mapper calls to compare against.

## Inputs

- invMap inversion-call output in SAM format (invMap_aligned.sam)
- competing-mapper inversion call sets (Winnowmap2, minimap2, NGMLR, lra)
- reference genome coordinates for interval overlap comparison
- simulated ground-truth inversion annotations (e.g., Supplementary Table S5, Figures S3-S4)

## Outputs

- evidence table (e.g., novel_inversions_evidence.csv) with columns for alignment identity, inversion length, supporting anchors, and ground-truth status
- list of invMap-only inversion calls with mean identity of non-ground-truth calls (e.g., 81.95%)

## How to apply

Load the invMap inversion-call outputs for the NA12878 real human genome sequencing dataset alongside the competing-mapper call sets (Winnowmap2, minimap2, NGMLR, lra). Compare call sets via interval overlap on the reference coordinates to isolate inversions predicted only by invMap. For each unique call, record alignment identity, inversion length (e.g., the 231 bp novel inversion absent from the simulated ground truth), and supporting anchors from the invMap chaining output. Cross-reference extracted calls with Supplementary Table S5 and Figures S3-S4 to label ground-truth-matching versus non-ground-truth calls and compute summary statistics such as the 81.95% mean identity of non-ground-truth invMap-only calls. Assemble records into a structured evidence table (e.g., novel_inversions_evidence.csv with identity, length, anchors, truth status).

## Related tools

- **invMap** (produces the inversion-call set (SAM alignment output with anchors) that is compared against competing mappers via interval overlap) — https://github.com/zhang134/invMap.git
- **PacBio** (long-read sequencing source whose reads span inversion breakpoints, generating the call sets compared by interval overlap)
- **Oxford Nanopore** (long-read sequencing source whose reads span inversion breakpoints, generating the call sets compared by interval overlap)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- The evidence table reproduces the novel 231 bp inversion entry detected only by invMap.
- The mean identity of non-ground-truth invMap-only calls equals the reported 81.95%.
- Winnowmap2, minimap2, NGMLR, and lra report no call overlapping the invMap-only interval on reference coordinates.
- Each invMap-only record carries alignment identity, inversion length, and supporting anchors from the chaining output.
- Ground-truth-matching versus non-ground-truth labels are consistent with Supplementary Table S5 and Figures S3-S4.

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.
- Detected calls may not belong to the simulated ground truth (e.g., the 231 bp novel inversion), so overlap analysis must label truth status explicitly.
- invMap 1.0.0 (2023/09/08) requires ~800 M disk space and minimum hardware such as Intel Core i3 2100; older Windows versions (Windows 7) are the documented baseline.

## Evidence

- [abstract] invMap can effectively find more candidate variant calls for inversions than the competing methods: "invMap can effectively find more candidate variant calls for inversions than the competing methods"
- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] The final mapping result is invMap_aligned.sam in SAM format.: "The final mapping result is invMap_aligned.sam in SAM format."
