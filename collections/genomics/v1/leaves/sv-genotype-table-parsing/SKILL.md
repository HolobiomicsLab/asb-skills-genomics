---
name: sv-genotype-table-parsing
description: Use when a published SV study deposits a curated callset (e.g. on Figshare)
  and you must reproduce its reported catalogue size and per-class composition from
  the deposited SV records, genotypes and copy numbers.
when_to_use_negative:
- The goal is de novo SV discovery from raw short-read alignments — use the four-caller
  consensus pipeline (Delly, Lumpy, Pindel, cn.MOPS + SURVIVOR) instead of parsing
  a deposited catalogue
- The deposited table is an unfiltered or intermediate callset (e.g. the 315 two-caller
  calls) rather than the final manually curated catalogue, so counts will not match
  the reported 113
- The input lacks per-record variant class annotations (SVTYPE), making per-class
  tallying impossible without re-annotation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SURVIVOR
  role: Its merge/stats/filter commands operate on the SV VCF format used for the
    curated catalogue and can summarize the callset before class-level tallying
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:35aae345f9294cde1cd5b7c96402818a83a6830456ca50b3aab4996b7a8ce985
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# SV genotype table parsing and catalogue composition verification

## Summary

Parses a deposited curated structural-variant catalogue (SV calls, genotypes and copy numbers) and tallies SV counts by class (deletions, duplications, inversions, translocations) to verify the published composition against the 113-SV fission yeast catalogue (23 deletions, 64 duplications, 11 inversions, 15 translocations).

## When to use

Use when a published SV study deposits a curated callset (e.g. on Figshare) and you must reproduce its reported catalogue size and per-class composition from the deposited SV records, genotypes and copy numbers.

## When NOT to use

- The goal is de novo SV discovery from raw short-read alignments — use the four-caller consensus pipeline (Delly, Lumpy, Pindel, cn.MOPS + SURVIVOR) instead of parsing a deposited catalogue
- The deposited table is an unfiltered or intermediate callset (e.g. the 315 two-caller calls) rather than the final manually curated catalogue, so counts will not match the reported 113
- The input lacks per-record variant class annotations (SVTYPE), making per-class tallying impossible without re-annotation

## Inputs

- Curated SV catalogue from the Figshare project (figshare.com/projects/fission_yeast_structural_variation/15798) with SV class, genotypes and copy numbers per strain
- Expected per-class composition (23 deletions, 64 duplications, 11 inversions, 15 translocations; total 113)

## Outputs

- sv_catalogue_composition.csv with per-class tallies, expected values, and match/mismatch flags
- Total record count and confirmation that the retrieved catalogue contains exactly 113 SV records

## How to apply

Download the curated structural-variant files (SV list with genotypes and copy numbers) from the Figshare project figshare.com/projects/fission_yeast_structural_variation/15798. Parse each record, capturing its variant class (deletion, duplication, inversion, translocation) plus genotype and copy-number fields, and tally counts per class. Compare per-class and total counts against the expected composition of 23 deletions, 64 duplications, 11 inversions and 15 translocations (total 113). Write tallies, expected values and match/mismatch flags to sv_catalogue_composition.csv. The two-call consensus filtering (SURVIVOR merge) plus manual inspection means the deposited set is a high-specificity curated catalogue, so any mismatch signals an error in retrieval or parsing rather than a discovery.

## Related tools

- **SURVIVOR** (Its merge/stats/filter commands operate on the SV VCF format used for the curated catalogue and can summarize the callset before class-level tallying) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
import pandas as pd; df = pd.read_csv('curated_sv_catalogue.tsv', sep='\t'); print(df['SVTYPE'].value_counts()); assert len(df) == 113
```

## Evaluation signals

- Retrieved catalogue contains exactly 113 SV records
- Per-class counts equal 23 deletions, 64 duplications, 11 inversions and 15 translocations
- All match/mismatch flags in sv_catalogue_composition.csv report a match against the expected composition
- Every parsed record carries class, genotype and copy-number fields with no unparseable rows

## Limitations

- The catalogue prioritized high specificity over high sensitivity, so its 113 SVs are not exhaustive and other SVs may exist in the strains
- Translocations and inversions were more challenging to genotype accurately and were not re-examined by the authors, so their class counts depend on the original manual curation
- Copy-number calls are only weakly correlated with nearby SNP variation (Spearman ρ=0.22, P=0.041), so copy-number fields cannot be sanity-checked against SNPs

## Evidence

- [abstract] Curation composition: "This curation produced a set of 113 SVs, comprising 23 deletions, 64 duplications, 11 inversions and 15 translocations"
- [discussion] Manual curation quality: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [readme] SURVIVOR scope on VCF callsets: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples"
- [discussion] Specificity over sensitivity: "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
