---
name: structural-variant-callset-retrieval
description: Use when a task requires obtaining the published curated SV catalogue
  (genotypes and copy numbers) for S. pombe strains and checking per-class counts
  against the paper's reported composition.
when_to_use_negative:
- Input is a raw set of SV predictions from a single caller (Delly, Lumpy, Pindel
  or cn.MOPS) rather than the deposited curated callset — the consensus/filtering
  pipeline would need to be rerun instead
- The task requires de novo SV discovery or genotyping of translocations/inversions;
  the paper explicitly did not re-examine genotypes of translocations and inversions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: SURVIVOR
  role: Tool set used to merge and compare caller VCFs into the consensus SV catalogue;
    its merge/filter/stats commands can assist parsing and verifying the retrieved
    callset
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: Delly
  role: One of the four SV inference packages whose calls were merged to form the
    curated catalogue being retrieved
- name: Lumpy
  role: One of the four SV inference packages whose calls were merged to form the
    curated catalogue being retrieved
- name: Pindel
  role: One of the four SV inference packages whose calls were merged to form the
    curated catalogue being retrieved
- name: cn.MOPS
  role: Copy-number caller whose population variant calls were manually curated to
    produce copy numbers in the deposited callset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:f63903b573860b53baf0d922e08609709edd9fc3161bbc43d79e75d0526e6628
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - Delly
  - Lumpy
  - Pindel
  - SURVIVOR
  - cn.MOPS
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# structural-variant-callset-retrieval

## Summary

Retrieve the deposited curated structural-variant callset for the worldwide S. pombe strain collection from the Figshare project (fission yeast structural variation, project 15798) and verify it reproduces the reported catalogue of 113 SVs (23 deletions, 64 duplications, 11 inversions, 15 translocations).

## When to use

Use when a task requires obtaining the published curated SV catalogue (genotypes and copy numbers) for S. pombe strains and checking per-class counts against the paper's reported composition.

## When NOT to use

- Input is a raw set of SV predictions from a single caller (Delly, Lumpy, Pindel or cn.MOPS) rather than the deposited curated callset — the consensus/filtering pipeline would need to be rerun instead
- The task requires de novo SV discovery or genotyping of translocations/inversions; the paper explicitly did not re-examine genotypes of translocations and inversions

## Inputs

- Deposited curated SV call files (VCF/tabular records with SV type, breakpoints, genotypes and copy numbers) from Figshare project fission_yeast_structural_variation/15798
- Reference expectation: reported composition of 23 deletions, 64 duplications, 11 inversions, 15 translocations (total 113)

## Outputs

- sv_catalogue_composition.csv with per-class tallies, expected values and match/mismatch flags
- Verified total count of 113 SV records

## How to apply

Download the curated SV call files (SV list with genotypes and copy numbers) from the Figshare project figshare.com/projects/fission_yeast_structural_variation/15798. Parse each SV record and classify it by variant class (deletion, duplication, inversion, translocation), tallying counts per class. Compare the tallies against the reported composition: 23 deletions, 64 duplications, 11 inversions, 15 translocations, total 113. Rationale: this catalogue was produced by a consensus approach (SURVIVOR merging of Delly, Lumpy, Pindel and cn.MOPS calls), rigorous filtering (SVs detected by at least two callers, removal of low-complexity/LTR-overlapping breakpoints), and manual vetting by visual inspection of read alignments, so the deposited records should exactly reproduce the published counts. Write the tallies, expected values and match/mismatch flags to an output file such as sv_catalogue_composition.csv.

## Related tools

- **SURVIVOR** (Tool set used to merge and compare caller VCFs into the consensus SV catalogue; its merge/filter/stats commands can assist parsing and verifying the retrieved callset) — https://github.com/fritzsedlazeck/SURVIVOR
- **Delly** (One of the four SV inference packages whose calls were merged to form the curated catalogue being retrieved)
- **Lumpy** (One of the four SV inference packages whose calls were merged to form the curated catalogue being retrieved)
- **Pindel** (One of the four SV inference packages whose calls were merged to form the curated catalogue being retrieved)
- **cn.MOPS** (Copy-number caller whose population variant calls were manually curated to produce copy numbers in the deposited callset)

## Examples

```
./SURVIVOR stats curated_svs.vcf  # summarize per-class counts of the deposited callset and compare with 23 DEL / 64 DUP / 11 INV / 15 BND
```

## Evaluation signals

- The retrieved catalogue contains exactly 113 SV records
- Per-class counts equal 23 deletions, 64 duplications, 11 inversions and 15 translocations
- Every parsed record is assigned to one of the four SV classes with no unclassified rows
- Record-level genotypes and copy numbers are present and non-null for the strains in the callset
- Match/mismatch flags in sv_catalogue_composition.csv are all 'match' against expected counts

## Limitations

- The study prioritized high specificity over high sensitivity, so not all SVs in the strains were detected and the catalogue is not exhaustive
- Translocation and inversion genotypes were not re-examined because they were more challenging to accurately genotype
- The stability of rearrangements was not examined by the authors, so copy numbers may not be stable across conditions

## Evidence

- [abstract] This curation produced a set of 113 SVs, comprising 23 deletions, 64 duplications, 11 inversions and 15 translocations: "This curation produced a set of 113 SVs, comprising 23 deletions, 64 duplications, 11 inversions and 15 translocations"
- [discussion] consensus approach followed by filtering and manual inspection: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [abstract] filtering to SVs detected by at least two callers (315 calls): "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [abstract] manual vetting by visual inspection of read alignments: "Finally, we manually vetted all the remaining SVs by visual inspection of read alignments in multiple strains for all remaining candidates."
- [readme] SURVIVOR description from README: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples, and includes various methods to reformat or summarize SVs."
