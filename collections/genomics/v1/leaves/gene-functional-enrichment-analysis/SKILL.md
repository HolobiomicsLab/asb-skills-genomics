---
name: gene-functional-enrichment-analysis
description: Use when you have two overlapping gene sets (e.g. coDIU and DIU genes
  from acorde) plus isoform-level functional annotations, and need to identify features/GO
  terms enriched in one set over the other.
when_to_use_negative:
- The gene sets to compare are non-overlapping — the partially-overlapping samples
  z-test is designed for sets that share genes
- You lack isoform-level positional functional annotations (IsoAnnotLite/tappAS .gff3),
  so no positional features can be tested
- The goal is isoform-level rather than gene-level functional analysis of alternative
  transcript expression
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: tappAS
  role: Runs the functional enrichment analysis with the partially-overlapping samples
    z-test on coDIU vs DIU genes using the functional annotation file
  repo: https://github.com/ConesaLab/tappAS
- name: acorde
  role: Produces the DIU/coDIU gene sets and the IsoAnnotLite-annotated long-read
    transcriptome used as enrichment input
  repo: https://github.com/ConesaLab/acorde
- name: IsoAnnotLite
  role: Transfers functional labels from tappAS pre-annotated mouse RefSeq references
    onto the long read-defined isoforms
  repo: https://isoannot.tappas.org/isoannot-lite/
- name: Revigo
  role: Removes redundant GO terms from the enriched list, reducing 59 GO terms to
    26 unique terms
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:670a7a68ca51fd35b00bd32a51463690a41084a905667eaef748dd914116777a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/tappAS
  tools:
  - IsoAnnotLite
  - R
  - Revigo
  - acorde
  - tappAS
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# gene-functional-enrichment-analysis

## Summary

Test whether positional functional features and GO terms are over-represented in coDIU genes relative to DIU genes using tappAS's partially-overlapping samples z-test, with Benjamini-Hochberg FDR correction at 0.05 and Revigo redundancy filtering.

## When to use

Use when you have two overlapping gene sets (e.g. coDIU and DIU genes from acorde) plus isoform-level functional annotations, and need to identify features/GO terms enriched in one set over the other.

## When NOT to use

- The gene sets to compare are non-overlapping — the partially-overlapping samples z-test is designed for sets that share genes
- You lack isoform-level positional functional annotations (IsoAnnotLite/tappAS .gff3), so no positional features can be tested
- The goal is isoform-level rather than gene-level functional analysis of alternative transcript expression

## Inputs

- IsoAnnotLite-annotated long-read transcriptome annotation (e.g. Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3)
- tappAS Mus musculus RefSeq functional annotation file
- Gene-to-condition table assigning genes to coDIU and DIU sets from the acorde DIU/coDIU analysis

## Outputs

- Table of positional functional features enriched in coDIU vs DIU genes with FDR values (91 features at FDR<0.05)
- Table of 59 GO terms enriched in coDIU vs DIU genes at FDR<0.05
- Revigo-filtered list of 26 unique (non-redundant) GO terms
- codiu_functional_enrichment.tsv

## How to apply

Load the long read-defined, IsoAnnotLite-annotated Mus musculus transcriptome produced by the acorde pipeline together with the tappAS mouse RefSeq functional annotation and the gene-to-condition table assigning genes to coDIU and DIU sets. In tappAS, run the functional enrichment analysis comparing coDIU against DIU genes using the partially-overlapping samples z-test, which is appropriate because the two gene sets share genes. Correct p-values across all tested features with Benjamini-Hochberg and retain features significant at FDR<0.05, which in the paper recovers 91 positional functional features and 59 GO terms. Finally, submit the enriched GO term list to Revigo to remove redundant terms, leaving 26 unique GO terms, and export results with FDR values (e.g. to codiu_functional_enrichment.tsv).

## Related tools

- **tappAS** (Runs the functional enrichment analysis with the partially-overlapping samples z-test on coDIU vs DIU genes using the functional annotation file) — https://github.com/ConesaLab/tappAS
- **acorde** (Produces the DIU/coDIU gene sets and the IsoAnnotLite-annotated long-read transcriptome used as enrichment input) — https://github.com/ConesaLab/acorde
- **IsoAnnotLite** (Transfers functional labels from tappAS pre-annotated mouse RefSeq references onto the long read-defined isoforms) — https://isoannot.tappas.org/isoannot-lite/
- **Revigo** (Removes redundant GO terms from the enriched list, reducing 59 GO terms to 26 unique terms)

## Evaluation signals

- Enrichment run recovers 91 positional functional features annotated in a significantly higher proportion in coDIU vs DIU genes at FDR<0.05
- Enrichment run recovers 59 GO terms enriched in coDIU vs DIU genes at FDR<0.05
- Revigo filtering of the 59 enriched GO terms yields exactly 26 unique GO terms
- p-values were corrected with the Benjamini-Hochberg procedure across all tested functional features
- Output file codiu_functional_enrichment.tsv contains the positional feature and GO term results including FDR values

## Limitations

- Functional annotation depends on IsoAnnotLite transfer from pre-annotated tappAS references; users must use compatible pre-annotated references until a de novo annotation tool is available
- A de novo functional annotation tool for long-read transcriptomes is currently a work in progress (tappAS GitHub issue #12)
- The reported functional insights are hypothesis-generating and require further experimental validation

## Evidence

- [other] Using the partially-overlapping samples z test, 91 positional functional features and 59 GO terms were found annotated in a significantly higher proportion in coDIU vs DIU genes (FDR < 0.05).: [evidence span withheld — anchor status: not_found]
- [readme] we used IsoAnnotLite (v2.6) to transfer functional labels from tappAS' pre-annotated mouse RefSeq 78 reference: [evidence span withheld — anchor status: not_found]
- [readme] use tappAS for functional analysis: "we incorporate functional annotations onto isoform models, and use [tappAS](https://github.com/ConesaLab/tappAS) for functional analysis"
- [intro] we incorporated a functional annotation step in which several databases and prediction tools were integrated to add isoform-specific functional information: "we incorporated a functional annotation step in which several databases and prediction tools were integrated to add isoform-specific functional information"
