---
name: figshare-data-deposit-verification
description: Use when a publication's central claim is a concrete count or composition
  (e.g. N variants per class) and the underlying call set is deposited at a public
  Figshare project URL, so the reported numbers must be reproduced from the deposit.
when_to_use_negative:
- You want to reproduce the raw SV discovery pipeline (Delly/Lumpy/Pindel/cn.MOPs
  consensus via SURVIVOR) rather than verify the final curated numbers in the deposit
- The deposit contains only raw caller output without the manual-curation annotations
  needed to assign each record to a class
- The claim to verify is a phenotype/GWAS association (e.g. 19 SVs significantly associated
  with traits) rather than a catalogue count directly tallyable from the call file
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:c8161d39ba580858d4758861338d3dc416b7e469b0fd99e532882541fa624997
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# figshare-data-deposit-verification

## Summary

Reproduces a paper's headline result by downloading its deposited data from a Figshare project and re-tallying the key reported quantities. Here, the curated S. pombe SV catalogue (113 SVs: 23 deletions, 64 duplications, 11 inversions, 15 translocations) is re-derived from the deposited call set with genotypes and copy numbers.

## When to use

Use when a publication's central claim is a concrete count or composition (e.g. N variants per class) and the underlying call set is deposited at a public Figshare project URL, so the reported numbers must be reproduced from the deposit.

## When NOT to use

- You want to reproduce the raw SV discovery pipeline (Delly/Lumpy/Pindel/cn.MOPs consensus via SURVIVOR) rather than verify the final curated numbers in the deposit
- The deposit contains only raw caller output without the manual-curation annotations needed to assign each record to a class
- The claim to verify is a phenotype/GWAS association (e.g. 19 SVs significantly associated with traits) rather than a catalogue count directly tallyable from the call file

## Inputs

- Figshare deposit files from figshare.com/projects/fission_yeast_structural_variation/15798: curated SV list with per-strain genotypes and copy numbers
- Published expected composition from the article (113 SVs; 23 deletions, 64 duplications, 11 inversions, 15 translocations)

## Outputs

- sv_catalogue_composition.csv containing per-class tallies, expected values, and match/mismatch flags
- Verified total SV count and composition (expected: 113 total)

## How to apply

Download the deposited curated SV call files — the list with genotypes and copy numbers — from the Figshare project figshare.com/projects/fission_yeast_structural_variation/15798. Parse each curated catalogue record and tally the counts per SV class (deletion, duplication, inversion, translocation). Compare each per-class tally and the total against the published composition: 23 deletions, 64 duplications, 11 inversions and 15 translocations, totalling 113. Write the tallies, expected values and match/mismatch flags to sv_catalogue_composition.csv so any discrepancy is explicit and auditable. Note that this catalogue came from a SURVIVOR consensus of Delly, Lumpy, Pindel and cn.MOPs calls followed by rigorous filtering and manual inspection, so the deposit represents the post-curation, not raw caller, output.

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('sv_catalogue.tsv',sep='\t'); counts=df['SVTYPE'].value_counts(); expected={'DEL':23,'DUP':64,'INV':11,'TRA':15}; pd.DataFrame({'tally':counts,'expected':pd.Series(expected),'match':counts==pd.Series(expected)}).to_csv('sv_catalogue_composition.csv')"
```

## Evaluation signals

- The parsed deposit contains exactly 113 SV records
- Per-class counts equal 23 deletions, 64 duplications, 11 inversions and 15 translocations
- Per-class counts sum to the total (23 + 64 + 11 + 15 = 113)
- Every record carries genotypes and copy numbers so class assignment is unambiguous
- All match/mismatch flags in sv_catalogue_composition.csv are 'match' when compared to the reported composition

## Limitations

- The catalogue prioritized high specificity over high sensitivity, so it is not an exhaustive list of all SVs in the strains — a count mismatch may reflect deposit versioning rather than curation error
- Translocations and inversions were not re-examined/genotyped during curation because they were more challenging to genotype accurately
- PCR and BLAST validation against de novo assemblies positively verified only 76% of rearrangements, so some deposited classes could still be mislabelled

## Evidence

- [abstract] A curated catalogue of 113 SVs (23 deletions, 64 duplications, 11 inversions, 15 translocations) was established in worldwide S. pombe strains: "This curation produced a set of 113 SVs, comprising 23 deletions, 64 duplications, 11 inversions and 15 translocations"
- [discussion] Curated calls emerged from a consensus plus filtering and manual inspection: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [abstract] Inversions and translocations were not re-genotyped, affecting record reliability by class: "Since translocations and inversions were more challenging to accurately genotype, we did not re-examine these variants."
- [abstract] Only 76% of rearrangements verified by PCR/BLAST against de novo assemblies: "Attempts to validate all rearrangements by PCR and BLAST searches of de novo assemblies positively verified 76% of the rearrangements"
