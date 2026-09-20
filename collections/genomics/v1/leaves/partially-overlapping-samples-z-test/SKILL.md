---
name: partially-overlapping-samples-z-test
description: Use when two gene sets being compared for functional feature annotation
  share a substantial fraction of members (e.g. coDIU and DIU gene sets from an acorde
  analysis), so a standard proportion test assuming independence is inappropriate.
when_to_use_negative:
- The gene sets being compared are fully disjoint, so a standard two-proportion enrichment
  test suffices and the partially-overlapping correction is unnecessary
- The transcriptome is de novo rather than a pre-annotated reference compatible with
  IsoAnnotLite/tappAS, since functional features cannot be transferred to the isoform
  models
- The goal is gene-level differential expression or DIU testing rather than functional
  annotation enrichment
edam_operation: http://edamontology.org/operation_2239
tools:
- name: tappAS
  role: Runs the functional enrichment analysis with the partially overlapping samples
    z-test and BH FDR correction on the coDIU vs DIU gene sets
  repo: https://github.com/ConesaLab/tappAS
- name: acorde
  role: Produces the DIU and coDIU gene sets and the IsoAnnotLite-annotated transcriptome
    that feed the enrichment test
  repo: https://github.com/ConesaLab/acorde
- name: Revigo
  role: Removes redundant GO terms from the enriched list, yielding 26 unique terms
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:eb110d24dc5ab39ee9f247861f1409c2947fb9c3d4887dad790658907ebf96d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/tappAS
  tools:
  - R
  - Revigo
  - acorde
  - tappAS
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# partially-overlapping-samples-z-test

## Summary

A tappAS statistical test for functional enrichment that compares the proportion of annotated genes between partially overlapping gene sets — used in the acorde study to identify positional functional features and GO terms enriched in coDIU versus DIU genes in mouse neural single-cell isoform data.

## When to use

Use when two gene sets being compared for functional feature annotation share a substantial fraction of members (e.g. coDIU and DIU gene sets from an acorde analysis), so a standard proportion test assuming independence is inappropriate.

## When NOT to use

- The gene sets being compared are fully disjoint, so a standard two-proportion enrichment test suffices and the partially-overlapping correction is unnecessary
- The transcriptome is de novo rather than a pre-annotated reference compatible with IsoAnnotLite/tappAS, since functional features cannot be transferred to the isoform models
- The goal is gene-level differential expression or DIU testing rather than functional annotation enrichment

## Inputs

- Long read-defined, IsoAnnotLite-annotated Mus musculus transcriptome annotation from the acorde pipeline (e.g. Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3)
- tappAS Mus musculus RefSeq functional annotation file
- Gene-to-condition table assigning genes to the coDIU and DIU sets

## Outputs

- Positional feature enrichment results (91 features enriched in coDIU vs DIU at FDR<0.05) with FDR values, e.g. codiu_functional_enrichment.tsv
- Enriched GO term list (59 terms at FDR<0.05), reducible to 26 unique terms after Revigo redundancy filtering

## How to apply

Load the long read-defined, IsoAnnotLite-annotated Mus musculus transcriptome produced by the acorde pipeline together with the tappAS Mus musculus RefSeq functional annotation and a gene-to-condition table assigning genes to the coDIU and DIU sets. Run tappAS functional enrichment comparing coDIU genes against DIU genes with the partially overlapping samples z-test, which accounts for gene membership shared between the two sets. Apply Benjamini-Hochberg correction across all tested functional features and retain features significant at FDR < 0.05. In the acorde study this recovered 91 positional functional features and 59 GO terms annotated in a significantly higher proportion in coDIU vs DIU genes; the GO term list was subsequently submitted to Revigo, reducing it to 26 unique terms.

## Related tools

- **tappAS** (Runs the functional enrichment analysis with the partially overlapping samples z-test and BH FDR correction on the coDIU vs DIU gene sets) — https://github.com/ConesaLab/tappAS
- **acorde** (Produces the DIU and coDIU gene sets and the IsoAnnotLite-annotated transcriptome that feed the enrichment test) — https://github.com/ConesaLab/acorde
- **Revigo** (Removes redundant GO terms from the enriched list, yielding 26 unique terms)

## Evaluation signals

- Output reproduces 91 positional functional features enriched in coDIU vs DIU genes at FDR < 0.05
- Output reproduces 59 GO terms enriched in coDIU vs DIU genes at FDR < 0.05
- After Revigo redundancy filtering, exactly 26 unique GO terms remain
- All reported p-values are Benjamini-Hochberg corrected across all tested functional features
- Enrichment table (codiu_functional_enrichment.tsv) includes per-feature FDR values

## Limitations

- The R package acorde does not include the long-read transcriptome building or functional analysis steps; they are run via external tools (IsoAnnotLite, tappAS)
- Functional annotation requires a pre-annotated reference compatible with IsoAnnotLite; a de novo annotation tool was still a work in progress (ConesaLab/tappAS issue #12)
- Functional insights require further experimental validation and are hypothesis-generating only

## Evidence

- [results] Partially-overlapping z-test recovery of 91 features / 59 GO terms: [evidence span withheld — anchor status: not_found]
- [readme] IsoAnnotLite annotation transfer from tappAS reference: [evidence span withheld — anchor status: external_doc]
- [intro] Functional annotation step feeding tappAS analyses: "we incorporated a functional annotation step in which several databases and prediction tools were integrated to add isoform-specific functional information"
- [readme] tappAS provides statistical methods for AltTEM analysis: [evidence span withheld — anchor status: external_doc]
