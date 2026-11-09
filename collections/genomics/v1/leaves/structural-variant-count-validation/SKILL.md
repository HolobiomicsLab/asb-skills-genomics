---
name: structural-variant-count-validation
description: Use when a downloaded SV callset (e.g. from the Figshare fission yeast
  structural variation project) must be checked against the published 113-SV catalogue
  of 23 deletions, 64 duplications, 11 inversions and 15 translocations before downstream
  analysis.
when_to_use_negative:
- You are re-calling or re-detecting SVs de novo from short-read data (use the Delly/Lumpy/Pindel/cn.MOPS
  + SURVIVOR consensus pipeline instead; this skill only validates an already-deposited
  catalogue)
- You need to re-validate breakpoint accuracy or re-genotype individual variants —
  the authors deliberately did not re-examine translocation and inversion genotypes
  because they were 'more challenging to accurately genotype'
- The callset under test is a different species or study, since the expected counts
  (113 SVs; 23/64/11/15) are specific to this S. pombe catalogue
tools:
- name: SURVIVOR
  role: 'Optional sanity check on the deposited VCF-style callset: its ''stats'' command
    reports summary statistics over a VCF of SVs to cross-check record counts and
    classes before tallying'
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:5539d64c4a643c6d1ec70d6a2733fc5638d5eb99b48552a7b7ef12760985f4b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# structural-variant-count-validation

## Summary

Validates that a deposited structural-variant catalogue reproduces the curated composition published for the worldwide S. pombe collection: exactly 113 SVs comprising 23 deletions, 64 duplications, 11 inversions and 15 translocations. It guards against silent corruption or re-curation drift in shared SV callsets.

## When to use

Use when a downloaded SV callset (e.g. from the Figshare fission yeast structural variation project) must be checked against the published 113-SV catalogue of 23 deletions, 64 duplications, 11 inversions and 15 translocations before downstream analysis.

## When NOT to use

- You are re-calling or re-detecting SVs de novo from short-read data (use the Delly/Lumpy/Pindel/cn.MOPS + SURVIVOR consensus pipeline instead; this skill only validates an already-deposited catalogue)
- You need to re-validate breakpoint accuracy or re-genotype individual variants — the authors deliberately did not re-examine translocation and inversion genotypes because they were 'more challenging to accurately genotype'
- The callset under test is a different species or study, since the expected counts (113 SVs; 23/64/11/15) are specific to this S. pombe catalogue

## Inputs

- Curated SV call files deposited on the Figshare fission yeast structural variation project (figshare.com/projects/fission_yeast_structural_variation/15798), including SV records with genotypes and copy numbers
- Expected composition reference: 23 deletions, 64 duplications, 11 inversions, 15 translocations (total 113)

## Outputs

- sv_catalogue_composition.csv containing per-class SV tallies, expected values, and match/mismatch flags
- Total record count confirming presence or absence of exactly 113 SVs

## How to apply

Download the curated SV list with genotypes and copy numbers from the Figshare project figshare.com/projects/fission_yeast_structural_variation/15798. Parse each SV record, extract its variant class (deletion, duplication, inversion, translocation) from the call format (e.g. the SVTYPE field of the VCF-style calls), and tally counts per class. Compare each per-class tally and the total against the published composition (23 deletions, 64 duplications, 11 inversions, 15 translocations, total 113), flagging each as match or mismatch. The rationale is that this catalogue was built by a consensus of Delly, Lumpy, Pindel and cn.MOPs calls merged with SURVIVOR, then rigorously filtered and manually vetted, so the deposited counts are the authoritative reference. Write the tallies, expected values and match/mismatch flags to sv_catalogue_composition.csv.

## Related tools

- **SURVIVOR** (Optional sanity check on the deposited VCF-style callset: its 'stats' command reports summary statistics over a VCF of SVs to cross-check record counts and classes before tallying) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
python -c "import collections,sys; c=collections.Counter(l.split('SVTYPE=')[1].split(';')[0] for l in open('sv_catalogue.vcf') if 'SVTYPE=' in l and not l.startswith('#')); print(dict(c)); assert sum(c.values())==113"
```

## Evaluation signals

- Total number of SV records equals exactly 113
- Per-class counts match 23 deletions, 64 duplications, 11 inversions and 15 translocations
- All match/mismatch flags in sv_catalogue_composition.csv are 'match'
- Every record carries class-defining annotations (e.g. SVTYPE) plus genotype and copy-number fields, with no unclassifiable entries
- Sum of per-class tallies reconciles with the total record count (23+64+11+15 = 113)

## Limitations

- The upstream catalogue prioritized high specificity over sensitivity, so not all SVs present in the strains were detected — a passing count check does not imply the catalogue is exhaustive
- Translocations and inversions were not re-examined/genotyped during curation because they are harder to genotype accurately, so class-level counts for these categories rest on less-validated calls
- Only 76% of rearrangements were positively verified (by PCR and BLAST searches of de novo assemblies), so some calls in the validated catalogue may still be incorrect

## Evidence

- [abstract] Published composition: "This curation produced a set of 113 SVs, comprising 23 deletions, 64 duplications, 11 inversions and 15 translocations"
- [discussion] Upstream pipeline provenance: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [abstract] Genotyping caveat: "Since translocations and inversions were more challenging to accurately genotype, we did not re-examine these variants."
- [discussion] Sensitivity limitation: "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
- [readme] SURVIVOR stats usage: [evidence span withheld — anchor status: external_doc]
